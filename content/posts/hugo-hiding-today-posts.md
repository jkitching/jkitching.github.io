---
title: "Hugo hides posts marked with current date"
date: 2024-06-05
---

I had uploaded a post marked with the current date to GitHub, which has an action set up to rebuild Hugo and post to GitHub's static page hosting.  The post didn't show.

I assumed it was because their server was in a different timezone (e.g. PST) and was interpreting the dates as such.  But then I loaded up a local server with `hugo server` and the post was not listed there either!

It turns out that Hugo has a setting in `config.toml` called [`timeZone`](https://gohugo.io/getting-started/configuration/#timezone).

According to an issue entitled [Add proper timezone support](https://github.com/gohugoio/hugo/issues/1882#issuecomment-195606655) on Hugo's GitHub tracker:

> 1. If timzeone *[sic]* set in the datetime string: fine, use that
> 2. If timezone set in config (or in page frontmatter), use that
> 3. Else use UTC

Well, that's pretty clear.  The post wasn't showing up because (a) I hadn't set a value for `timeZone`, and (b) it was still yesterday in UTC.

Adding a timezone has solved all of my woes:

```toml
timeZone = "Asia/Taipei"
```

# Bonus: showing future-dated posts

I almost never write multiple posts in a single day, so my frontmatter only includes the date with `YYYY-MM-DD` granularity.  I would prefer to spread my posts out to at most one per day, so the solution that I opt for is to future date (e.g. second post of the day gets dated as tomorrow).

But future-dated posts are hidden by default!  Perhaps that's a good idea for dynamically generated websites... but for something like Hugo, who is going to be sitting around rebuilding when those future dates arrive?

An easy hack is to add the `--buildFuture` argument when invoking `hugo`.  After adding the flag to `.github/workflows/gh-pages.yaml` in this repository, all of my dedicated blog readers can now live in the future. =)
