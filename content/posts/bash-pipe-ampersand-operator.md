---
title: "The |& operator: piping stderr since 1978"
date: 2025-12-23
---

When running a command, it's often useful to save output to a file while also displaying it on the terminal.  The `tee` command handles this elegantly:

```sh
./my-script.sh | tee output.log
```

But this only captures stdout.  To also capture stderr, the traditional approach is:

```sh
./my-script.sh 2>&1 | tee output.log
```

I recently learned about the fancy shorthand operator `|&`, which does the same:

```sh
./my-script.sh |& tee output.log
```

This implicitly redirects stderr to stdout, then pipes the combined stream to `tee`.  Both streams appear on the terminal and in `output.log`.

Ridiculously cool.  But we must go deeper!  Where did this operator come from?

## Bill Joy and csh

The `|&` syntax predates Bash by decades.  It originated in the C shell (csh), created by Bill Joy at UC Berkeley and first distributed with 2BSD in 1978.  Unlike stdout, which could be redirected with `>`, csh had no way to redirect stderr alone---so `|&` was the only option for piping error output.

## The ksh conflict

The syntax went unchallenged for a decade.  Then in 1988, ksh (the Korn shell) claimed the same `|&` for a completely different purpose: starting a *coprocess*.  Now the same two characters mean completely different things depending on your shell.

> A **coprocess** is a background process with a bidirectional pipe---your script can both write to its stdin and read from its stdout.  Handy for interactive programs like calculators or REPLs:
> 
> ```sh
> bc |&  # start bc as a coprocess
> print -p 'scale=3'
> print -p '5 / 3'
> read -p result  # result = 1.666
> ```

## Bash picks a side

When Bash 4.0 arrived in 2009, it introduced `|&` with csh's semantics (pipe stderr+stdout together) and added a separate `coproc` keyword for ksh-style bidirectional pipes.  zsh made the same choice.

## Should you use it?

Of course!  It's short and convenient.  Although I still seems to bugger up the order of those two characters half the time.  The `2>&1 |` form is more portable and universally understood, so perhaps still better to use in a shared codebase.

See also: [How do I write standard error to a file while using "tee" with a pipe?](https://stackoverflow.com/questions/692000/how-do-i-write-standard-error-to-a-file-while-using-tee-with-a-pipe) on Stack Overflow.
