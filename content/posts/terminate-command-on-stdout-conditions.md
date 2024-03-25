---
title: "Terminating a command early on matching stdout conditions"
date: 2024-03-25
---

# Match and exit

This [StackExchange question](https://unix.stackexchange.com/questions/679658/kill-process-once-it-produces-certain-output) provides a snippet which runs a subprocess, saves its PID, and then ends the process when a certain output string is encountered in its output.

```sh
sh -c 'echo "$$"; exec stdbuf -oL "$0" "$@"' my-program with its args | (
  IFS= read -r pid &&
    sed '/Was exported to:/q' &&
    kill -s PIPE "$pid"
)
```

# Match and exit, with short-circuit

I encountered a situation in which I wanted to exit early when it was determined that the matching condition would never be met.

The output of `my-program` is a list of newline-separated JSON objects, so each line needs to be processed individually, checking these two conditions:

1. Success, which should output the entire line (matching JSON object)
2. Short-circuit failure, which should not output any text (no match)

```sh
sh -c 'echo "$$"; exec stdbuf -oL "$0" "$@"' my-program with its args | (
    IFS=
    read -r pid
    while read -r line; do
        # Break when the condition occurs, and output the matching JSON object
        if echo "$line" | jq -e "select(.key == \"value\")" > /dev/null; then
            echo "$line"
            break
        fi
        # Break when it is determined that the condition cannot be met
        if ! echo "$line" | jq -e "select(.key == \"value\")" > /dev/null; then
            break
        fi
    done
    kill -s PIPE "$pid" 2>/dev/null
)
```

Here, `jq`'s `-e` flag will set a failing exit status if the `select()` function fails.

To output the JSON object in the matching case, `echo "$line"` is run before breaking out of the loop.

# Shorter conditionals with logical operators

We can also shorten the conditionals by taking advantage of the fact that `select()` will output the matching object:

```sh
# Break when the condition occurs, and output the matching JSON object
echo "$line" | jq -ce "select(.key == \"value\")" && break
# Break when it is determined that the condition cannot be met
echo "$line" | jq -e "select(.key == \"value\")" > /dev/null || break
```

`jq`'s `-c` flag ensures that the matching object is printed on one line.  The matching case no longer redirects to `/dev/null` since we want the output, and the `!` negation in the short-circuit case results in using a logical `||` instead of `&&`.
