---
title: "Restarting a process when a signal is received"
date: 2024-04-10
---
In larger applications, there is usually a CI/CD process, which tests, builds, and deploys new versions of the software.

Smaller applications typically start out with a simpler solution.  In this case, I have an application running in a `screen` session, and I want a git `post-receive` hook to automatically [update its code]({{< ref "git-clean-deployment-hook" >}}) and restart the process.

**Is it possible to write a one-line Bash wrapper which restarts the application when a specific signal is received?** [TL;DR: Yes, it is possible.](#using-exec-as-a-one-liner)

This particular application is not thoroughly robust yet, so I do not want to restart it when the process exits on its own.  Thus we have the following requirements:

* When the application exits, the wrapper quits with its exit code
* When the wrapper is killed with SIGHUP, restart the application
* When the wrapper is killed with any other signal, kill the application and exit

Note that since the final goal is to write a compact one-liner, quotes will be omitted where possible.

# Tail recursive approach

In this version, the `run` call in the `HUP` trap essentially takes over execution of the process.  When it pops up to the previous trap, `exit` stops execution and propagates the exit code.

```sh
pid=
run() {
    echo restarting
    sleep inf &
    pid=$!
    wait $pid
}
trap 'kill $pid 2>/dev/null' EXIT
trap 'kill $pid; run; exit $?' HUP
run
```

Note that we need the extra `EXIT` trap to handle the case of killing the wrapper directly.  Let's test sending various signals to the parent of `sleep` as well as the `sleep` process itself:

```sh
# Send HUP and TERM signals to child process
kill -HUP $(pgrep -f 'sleep inf')                      # exits
kill $(pgrep -f 'sleep inf')                           # exits

# Send HUP and TERM signals to wrapper
kill -HUP $(ps -o ppid= -p "$(pgrep -f 'sleep inf')")  # restarts
kill $(ps -o ppid= -p "$(pgrep -f 'sleep inf')")       # exits

# Send INT signal to wrapper's process group
<Ctrl+C> directly on the shell                         # exits
kill -INT -- -$(ps -o pgid= -p $(ps -o ppid= -p $(pgrep -f 'sleep inf')))
```

Everything checks out.  Next, we will stress-test this script, spamming it with `HUP` signals:

```sh
wrapperpid=$(ps -o ppid= -p "$(pgrep -f 'sleep inf')")
count=0
while true; do
    echo $((count++))
    kill -HUP $wrapperpid || break
    # Wait for wrapper to finish restarting child process
    # Also try without this sleep call after fixing bugs listed below
    sleep 0.01
done
```

It turns out this invariably causes the script to segfault, since it isn't true tail-optimized recursion---a stack overflow occurs somewhere around the 2800th restart on my machine.

Additionally, multiple `sleep` processes may end up running at the same time during execution, and there may be multiple `sleep` processes hanging around after the segfault occurs (or after killing the wrapper manually).  There are two independent problems at play; can you find them?  (Hmm, this seems like a great interview question...)

1. The first occurs in the `run()` function, after starting a new `sleep` process, and before recording its PID: `sleep inf & pid=$!`.  If the trap is triggered here, the PID will still point to the old `sleep` process and the new one will not be killed.  The only way to solve this is to disable the trap during this critical section.

2. The second occurs within the `SIGHUP` trap itself: `kill $pid; run`.  `kill` simply delivers a signal to the process, but does not wait for it to end.  Since `run` goes ahead and starts a new `sleep` process, there may be multiple `sleep` calls running simultaneously.  That said, given enough time, the extra (old) instances will eventually exit.  Testing this is therefore slightly tricky.  I replaced `sleep inf` with `( trap 'sleep 1' EXIT; while true; do true; done )` to confirm the problem.  We can solve this issue by inserting a `wait $pid` in between `kill` and `run`.

Here's a slightly more "correct" version of the script which takes these two issues into account.  Unfortunately, there is nothing we can do about the stack overflow without a rewrite.

```sh
pid=
trap 'kill $pid 2>/dev/null' EXIT
run() {
    echo restarting
    trap '' HUP
    sleep inf &
    pid=$!
    trap 'kill $pid; wait $pid; run; exit $?' HUP
    wait $pid
}
run
```

# Infinite loop approach

In this approach, an infinite while loop continually waits for the process to exit.  When this happens, check whether or not the `sighup` flag was set in the `SIGHUP` trap.  If it was, restart the application.  Otherwise, exit the while loop.

```sh
pid=
sighup=1
trap 'kill $pid 2>/dev/null' EXIT
trap 'sighup=1; kill $pid' HUP
while [ "$sighup" -eq 1 ]; do
    sighup=0
    echo restarting
    sleep inf &
    pid=$!
    wait $pid
done
```

This iterative approach contains two particularly racy conditions.  Can you find them?  They are both quite similar to those encountered in the recursive approach.

1. The first is after starting a new `sleep` process, and before recording its PID: `sleep inf & pid=$!`.  This can be solved by disabling the trap during this critical section.

2. The second is slightly more subtle.  When Bash receives a signal while running a built-in command like `wait`, it interrupts the command, runs the signal, and then continues to the command right *after* the one interrupted.  In this case, if `wait $pid` is interrupted by `SIGHUP`, execution continues after the `wait $pid` command.  So there is no guarantee the process has been killed yet.  To fix this, we add a `wait $pid` at the end of the `SIGHUP` trap.

Here's the updated version:

```sh
pid=
sighup=1
trap 'kill $pid 2>/dev/null' EXIT
while [ "$sighup" -eq 1 ]; do
    sighup=0
    echo restarting
    trap '' HUP
    sleep inf &
    pid=$!
    trap 'sighup=1; kill $pid; wait $pid' HUP
    wait $pid
done
```

Unfortunately, it turns out this version also suffers from the segfault issue.  Again, my understanding is that if the wrapper gets spammed with `HUP` signals, the traps will repeatedly get interrupted, and there will be a huge call stack of traps that eventually exceed the size of the available stack.

Perhaps we can try to fix this by making the trap just one statement, so that if it is interrupted, there will be nothing to run afterwards?  Will Bash be smart enough to optimize these "tail recursions"?

```sh
pid=
sighup=1
trap 'kill $pid 2>/dev/null' EXIT
while [ "$sighup" -eq 1 ]; do
    sighup=0
    echo restarting
    sleep inf &
    pid=$!
    trap 'sighup=1' HUP
    wait $pid
    trap '' HUP
    kill $pid 2>/dev/null
    wait $pid
done
```

Note the careful order of `trap`, `kill`, `wait`.

This version results in Bash interspersing our application restarts with "bad value" warnings.  I believe this is related to traps getting interrupted before running to completion, but I was unable to find a viable solution.

```sh
bash: warning: run_pending_traps: bad value in trap_list[1]: 0x1
```

But there is no segfault.  Rejoice---this is our most stable version so far!

There is actually one more potential issue which exists in this solution and the recursive solution.  If the wrapper process is killed in between starting the application and recording its PID (`sleep inf & pid=$!`), the child process will continue.  I have not been able to reproduce this behaviour without inserting a pause in between these two commands, but I have also not proven that it cannot occur.

This is a useful and viable version to use, but let's also explore one more option.

# Replacing wrapper using exec

This strategy replaces the running process with another by using `exec`.  It unfortunately requires running the script as a file.  [Or does it?  Heh heh heh...](#using-exec-as-a-one-liner)

```sh
#!/bin/bash

echo restarting
trap 'kill $pid 2>/dev/null' EXIT
sleep inf &
pid=$!
trap 'kill $pid 2>/dev/null; wait $pid; exec "$0" "$@"' HUP
wait $pid
```

The first iteration segfaults when spammed with `SIGHUP`.  Presumably traps are building up in the call stack before they finish executing until the call stack runs out of memory.

Additionally, there is a problem where if the `HUP` trap hasn't been installed yet, the script will just exit when it receives `SIGHUP`.  That is the default behaviour of `SIGHUP`---and it is actually quite hard to find a signal *without* any default behaviour.  I ended up choosing `SIGURG`:

> The operating system sends this signal to a process using a network connection when "urgent" out of band data is sent to it.

I am going to cut this journey short and say that I tried every possible permutation, and none of them worked.  So I turned this model on its head: let the replacement process handle killing the application before spawning a new one:

```sh
#!/bin/bash

trap 'pkill -P $$' EXIT
pkill -P $$
pidwait -P $$
echo restarting
sleep inf &
trap 'exec "$0" "$@"' URG
wait $!
```

This is the version that worked the best, with one small caveat: if the wrapper is killed before the `EXIT` trap is installed, the application process will continue running.

If this is a concern for you, consider using the infinite loop approach instead.  If you would like to play with quines, read onward!

# Using exec as a one-liner

Ever heard of a [quine](https://en.wikipedia.org/wiki/Quine_(computing))?  It's a program that, when run, will output its own source code.

Instead of a program outputting its source code, let's write a program which *runs* its own source code.  Behold:

```sh
( export program="echo run myself; eval \"\$program\""; eval "$program" )
```

This `$program`, once invoked, will respawn itself indefinitely.  Or rather, until it segfaults.  Let's try again!

```sh
( export program="echo run myself; exec bash -c \"\$program\""; eval "$program" )
```

Behold!  It really does respawn indefinitely.  Pretty neat, huh?  Now we shove everything from our script above into the string, and we get:

```sh
( export wrapper="trap 'pkill -P \$\$' EXIT;
  pkill -P \$\$; pidwait -P \$\$;
  echo restarting; sleep inf &
  trap 'exec bash -c \"\$wrapper\"' URG;
  wait \$!"; exec bash -c "$wrapper" )
```

And that's all there is to it. =)

# Identifying the wrapper

Up until now, we have swept the problem of identifying the wrapper PID under the carpet, relying on a crude method of finding the parent PID of the `sleep inf` process.

There are two options here.  The first is to save the wrapper's PID in a file:

```sh
( export wrapper="trap 'pkill -P \$\$' EXIT;
  pkill -P \$\$; pidwait -P \$\$;
  echo restarting; sleep inf &
  trap 'exec bash -c \"\$wrapper\"' URG;
  wait \$!"; exec bash -c "echo \$\$ > wrapper.pid; $wrapper" ); rm wrapper.pid

# Ask wrapper to restart the application:
kill -URG $(cat wrapper.pid)
```

The second is to assign a custom name to the process, which `exec` conveniently provides as the `-a` option:

```sh
( export wrapper="trap 'pkill -P \$\$' EXIT;
  pkill -P \$\$; pidwait -P \$\$;
  echo restarting; sleep inf &
  trap 'exec -a app_wrapper bash -c \"\$wrapper\"' URG;
  wait \$!"; exec -a app_wrapper bash -c "$wrapper" )

# Ask wrapper to restart the application:
pkill -f -URG app_wrapper
```

# Caveat

If you have made it this far, you may have tried replacing `sleep inf` with a script or a function.  It's remarkably hard to reliably kill all subprocesses when a parent process is killed.  Consider this example:

```sh
$ { sleep 100; echo done; } &
[1] 688046
$ kill 688046
$ ps ax | grep 'sleep 100'
 688047 pts/10   S      0:00 sleep 100
```

To deal this complexity, consider simply killing the entire process group (like Ctrl+C does): `kill -- -process_group` (note the extra dash).  If you would like to deal with this directly from the wrapper, `set -m` might be helpful.  It temporarily enables Bash jobs control, which causes every child process to be created in its own separate process group.

For example, modifying the infinite loop version of the wrapper to handle creating process groups might look like this:

```sh
pid=
sighup=1
trap 'kill $pid 2>/dev/null' EXIT
while [ "$sighup" -eq 1 ]; do
    sighup=0
    echo restarting
    set -m
    sleep inf &
    set +m
    pid=$!
    trap 'sighup=1' HUP
    wait $pid
    trap '' HUP
    kill -- -$pid 2>/dev/null
    wait $pid
done
```

Modifying the exec approach might look like this:

```sh
#!/bin/bash

trap 'pgrep -P $$ | xargs -I {} kill -- '-{}' 2>/dev/null' EXIT
pgrep -P $$ | xargs -I {} kill -- '-{}' 2>/dev/null
pidwait -P $$
echo restarting
set -m
sleep inf &
set +m
trap 'exec "$0" "$@"' URG
wait $!
```

Or, keep that complexity in your application function/script:

```sh
target() (
    trap 'kill -- -$pid 2>/dev/null' EXIT
    set -m
    sleep inf &
    pid=$!
    wait $pid
)
```

# Addendum

Remember that the above scripts will keep running until the application decides to exit on its own.  If you would like a restart to also occur in that case, then I shall leave that modification as an exercise to the reader.

Finally, also consider just sending `SIGHUP` to your application, and having the wrapper restart it based on its exit code.  This is vastly simpler since traps aren't needed:

```sh
while true; do
    echo restarting
    sleep inf
    [ $? -eq 129 ] || break
done
```

I had tons of fun learning about Bash traps while writing this article.  For more information, check out:

* [GreyCat's wiki---SignalTrap](https://mywiki.wooledge.org/SignalTrap)
* [StackExchange---explanation of WCE (wait and cooperative exit)](https://superuser.com/questions/1829830/behavior-of-sigint-with-bash)
