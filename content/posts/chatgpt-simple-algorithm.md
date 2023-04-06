---
title: "Coding a simple algorithm with ChatGPT"
date: 2023-04-06
draft: false
---

Lately, it's almost impossible to open up Hacker News without encountering an article extolling the virtues of LLM-assisted programming.  I wanted to give it a try---but I decided to start with something small.

# Problem definition

> Given an input array of 0's and 1's, produce an array where `output[i]` gives the consecutive count of `input[i]` up until position `i`.
> 
> For example:
> 
> * `1110100001` &rarr; `1231112341`
> * `1010101` &rarr; `1111111`
> * `1111111` &rarr; `1234567`
> 
> Write a Python function called `consecutive_count`, which takes a list as its input, and outputs a list.

# Attempt #1

Fairly simple, right?  I feed this text verbatim into ChatGPT, and it gives me this function:

> ```python
> def consecutive_count(arr):
>     n = len(arr)
>     output = [0] * n
>     count = 0
>     for i in range(n):
>         if arr[i] == 0:
>             count = 0
>         else:
>             count += 1
>         output[i] = count
>     return output
> ```

Looks like it's counting consecutive 1's, resetting the count to 0 when it encounters a 0.  This is confirmed by both ChatGPT's explanation, and the example it provides:

> ```python
> >>> consecutive_count([1,1,1,0,1,0,0,0,0,1])
> [1, 2, 3, 0, 1, 0, 0, 0, 0, 1]
> ```

# Attempt #2

Perhaps my explanation is not clear enough for ChatGPT to interpret the correct meaning?  I try a variation of the above prompt:

> Given an input array of arbitrary elements, produce an array where `output[i]` gives the consecutive count of `input[i]` up until position `i`, starting at 1.
> 
> For example:
> 
> * 1110100001 &rarr; 1231112341
> * 1010101 &rarr; 1111111
> * 1111111 &rarr; 1234567
> 
> Write a Python function called `consecutive_count`, which takes a list as its input, and outputs a list.

This time, ChatGPT seems to think that it should output the *overall* count of each encountered item, without resetting the count after a contiguous group ends:

> ```python
> def consecutive_count(input_list):
>     count = {}
>     output_list = []
>     for element in input_list:
>         if element not in count:
>             count[element] = 0
>         count[element] += 1
>         output_list.append(count[element])
>     return output_list
> ```

But the example input/output it provides doesn't quite match this algorithm, leading me to believe that its internal representation of the problem is inconsistent.

# Attempt #3

Perhaps we're overloading ChatGPT with too much information?  Let's eliminate specific references to 0 and 1 in the prompt, avoid algebraic terms like `input[i]`, and do away with the examples.

> Given an input list of arbitrary items, produce a list of equivalent length, which provides the consecutive count of equivalent items up until that point in the input list.  Counts start at 1.
> 
> Implement this with a Python function called `consecutive_count`.

Success! This time, it spits out a working function:

> ```python
> def consecutive_count(lst):
>     if not lst:
>         return []
>     counts = [1]
>     prev_item = lst[0]
>     for item in lst[1:]:
>         if item == prev_item:
>             counts.append(counts[-1] + 1)
>         else:
>             counts.append(1)
>             prev_item = item
>     return counts
> ```

It's a little on the verbose side though.  Let's try:

> Rewrite `consecutive_count` to use less temporary variables.

> ```python
> def consecutive_count(lst):
>     counts = [1] * len(lst)
>     for i in range(1, len(lst)):
>         if lst[i] == lst[i - 1]:
>             counts[i] = counts[i - 1] + 1
>     return counts
> ```

That is much more concise!  Thank you, ChatGPT.

# Enter functional programming

Let's see if ChatGPT is fluent in functional programming:

> Rewrite `consecutive_count` to use functional programming.

> ```python
> from itertools import groupby
> 
> def consecutive_count(input_list):
>     groups = groupby(input_list)
>     return [sum(1 for _ in group) for _, group in groups]
> ```

Well... that's not quite right.  This function outputs the size of each contiguous group of equivalent items.

> That's not quite right!  This function just outputs the sizes of each group, instead of counting up from 1 within each group.

> ```python
> from itertools import groupby
> 
> def consecutive_count(input_list):
>     output_list = []
>     for _, group in groupby(input_list):
>         group_size = len(list(group))
>         output_list += list(range(1, group_size + 1))
>     return output_list
> ```

That's better!  But a bit on the verbose side.

> That looks good, but avoid using a `for` loop.

> ```python
> from itertools import groupby
> 
> def consecutive_count(lst):
>     return [j for _, g in groupby(lst) for j in range(1, len(list(g))+1)]
> ```

Not bad at all.  However, some people (such as myself) find double list comprehensions somewhat difficult to read.

> That looks fantastic.  Could we break the list comprehension into two steps?

> ```python
> from itertools import groupby, chain
> 
> def consecutive_count(input_list):
>     groups = (list(g) for _, g in groupby(input_list))
>     return list(chain.from_iterable(range(1, len(g) + 1) for g in groups))
> ```

Wow!  I actually had to look up [`chain.from_iterable`](https://docs.python.org/3/library/itertools.html#itertools.chain.from_iterable) to see what was going on here.  It's basically a generator-aware flatten function, which is just what we need.

# Alternative solution

Originally, I had a solution in mind which uses `reduce`.  Let's see if we can guide ChatGPT to use this method:

> Please rewrite `consecutive_count` to make use of `reduce`.

> ```python
> from itertools import groupby
> from functools import reduce
> 
> def consecutive_count(lst):
>     groups = map(lambda x: list(x[1]), groupby(lst))
>     counts = map(lambda x: range(1, len(x) + 1), groups)
>     return reduce(lambda acc, x: acc + list(x), counts, [])
> ```

This is not quite what I was looking for... it basically uses `reduce` as a glorified `map`.  Let's try again.

> Please rewrite `consecutive_count` to make use of the `reduce` function.  There should be no need to use `groupby` or `map`.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     def accumulate_counts(acc, item):
>         if acc and acc[-1][0] == item:
>             acc[-1] = (item, acc[-1][1] + 1)
>         else:
>             acc.append((item, 1))
>         return acc
>     return [count for _, count in reduce(accumulate_counts, lst, [])]
> ```

This is actually really close to a working implementation.  For each item encountered, a tuple is added to the accumulator output list.  The first element of the tuple is the *current item* (the value at `lst[i]`), and the second element is the *count* (the output of our algorithm).  Finally, the first element of each tuple is stripped off in a final list comprehension.

But for some reason, ChatGPT is *modifying* the last element of the accumulator output list in the `if` clause of the conditional.

> That looks really close.  But the `if` clause of the conditional should be *appending* to `acc` instead of modifying its last value.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     def accumulate_counts(acc, item):
>         if acc and acc[-1][0] == item:
>             acc.append((item, acc[-1][1] + 1))
>         else:
>             acc.append((item, 1))
>         return acc
>     return [count for _, count in reduce(accumulate_counts, lst, [])]
> ```

Wow!  We did it!  Now let's iterate for style:

> Great!  Can you factor out the `append` call?  The second element of the tuple passed to `append` should be a conditional expression.  Assume that `acc` has at least one element in it, so there is no need to check for `acc` in the conditional.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     def accumulate_counts(acc, item):
>         count = (acc[-1][1] + 1) if acc[-1][0] == item else 1
>         acc.append((item, count))
>         return acc
>     return [count for _, count in reduce(accumulate_counts, lst, [(None, None)])][1:]
> ```

Almost there!

> Awesome.  Let's turn `accumulate_counts` into a lambda and assign it to a variable.  Call it `acc_counts` instead.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     acc_counts = lambda acc, item: (acc + [(item, (acc[-1][1] + 1) if acc[-1][0] == item else 1)])
>     return [count for _, count in reduce(acc_counts, lst, [(None, None)])][1:]
> ```

And we're done.

# Extra points

Let's explore how nimble ChatGPT is on its feet.

> I'd like to modify this `reduce` implementation to store *only one* instance of the current item.  The accumulator data format should look like: `(current_item, [output_list])`.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     acc_counts = lambda acc, item: (item, acc[1] + ([1] if item != acc[0] else [acc[1][-1]+1]))
>     return acc_counts[1] if lst else []
> ```

Subscripting a lambda... that's... cute?  Two more prompts to get ChatGPT back on track:

> That looks close, but you forgot to run `reduce`!

> There's no need for the conditional on the last line checking `lst`.

> ```python
> from functools import reduce
> 
> def consecutive_count(lst):
>     acc_counts = lambda acc, item: (item, acc[1] + ([1] if item != acc[0] else [acc[1][-1]+1]))
>     return reduce(acc_counts, lst, (None, []))[1]
> ```

And we did it!  ChatGPT, you are certainly impressive.

# Conclusion

I have a confession to make: this process was nowhere near as simple as this blog post makes it sound.

* Initially, I went down a huge rabbit hole of using the initial definition of the problem (specifying 0's and 1's).  I asked for an implementation, copied errors or incorrect output from Python back into the ChatGPT prompt, and repeated this process about 10 times.  ChatGPT didn't seem to grasp the concept, providing many variations, none of which were correct.
* Then, I tried to "teach" ChatGPT how to perform the algorithm step-by-step.  But once we reached the conditional inside the accumulator function (we were working on the `reduce` implementation), ChatGPT was **convinced** that the last value of the accumulator list needed to be *replaced* instead of appending a new tuple.  **NOTHING** I said could convince it otherwise.
    > ```python
    > if acc and acc[-1][0] == item:
    >     acc[-1] = (item, acc[-1][1] + 1)
    > else:
    >     acc.append((item, 1))
    > ```
* At this point, I began to doubt whether ChatGPT even understood the problem in the first place.  I prompted it with the problem definition, provided several input/output examples, and asked it to generate some examples on its own.  Many of the examples that ChatGPT generated were incorrect.  I kept fixing its examples and prompting them back, hoping that its internal representation would eventually align.  But this approach was definitely a dead end!  In fact, sometimes I would fix an example, and ChatGPT would make slight modifications when acknowledging and repeating the example back to me.
* Finally, I tried the approach that ultimately worked for this blog post: rewriting the problem definition to be more general, without specifying the input constraints of using only 0's and 1's.  I suspect that this algorithm may exist in ChatGPT's training data, but perhaps without the constraint of using only 0's and 1's.  Maybe once that constraint was removed, ChatGPT had an easier time mapping to the correct answer internally.

So how does it feel to write code with your own personal assistant?  Initially, it felt maddening, like talking to a very smart child who could implement complex algorithms, but refused to make fixes to said algorithm when asked.

Once I got past the initial difficulty of properly communicating the problem, and started to get the hang of prompting, it was actually quite a rewarding experience.  I could be as vague or specific as I wanted, and ChatGPT was generally very accommodating.  Of course, there were still certain cases where I had to revise my prompt.  For example, when asking ChatGPT to write a **slightly** more concise version, it might go from eight lines directly to a one-liner.  Prompting is still somewhat of an art.

It also strangely felt like ChatGPT and I were working as a team, which is why I keep falling back to the "we" pronoun in this blog post.  I found myself politely asking ChatGPT for help, and praising it (him? her? them?) for good answers.

I initially set out on this adventure specifically with the `reduce` solution in mind.  But I actually prefer the solution that ChatGPT invented, which uses `groupby` and `from_iterable`.  The fact that ChatGPT can understand the "gist" of what I'm trying to accomplish, and effortlessly translate between different implementations, is infinitely cool.  And also provides for learning opportunities!

I don't have access to GPT-4 yet, but I would be very curious to see how it performs with the initial problem definitions given in [Attempt #1](#attempt-1) and [Attempt #2](#attempt-2).  I invite any happy readers to paste those prompts into GPT-4 and let me know what happens!
