---
title: "Amazon intern day 12: in which a project is chosen"
date: 2011-05-11
---

I had a fairly decent sleep, and woke up to... a Quinn's head in between my bedroom door and the bathroom door.  I had to carefully step over it a few times to get in between the two rooms.  I showered, and quietly ate some breakfast.

By the time I was about to leave, Quinn had gotten up and was getting ready to take the early bus in (despite having claimed the night prior that he would be staying until the girls left in the afternoon).  So, without even saying goodbye to Mariana (who had just woken up) and Monica, he came with me downstairs, we said goodbye, and he headed off in his own direction!

# 2pt: two pizza team

Today at work we had a Two Pizza Team meeting.  What is a Two Pizza Team you ask?  It's a group of people small enough to feed with two pizzas.  Historically, all of Prime used to be included in a 2pt, but then it grew much too large to be fed by merely two pizzas.  The purpose of a 2pt is to cross-cut different positions (engineers, marketers, testers) in a small, agile team that can choose a little project to work on and go at it with little interference.  It's sort of Google 20% time-esque, except more organized into a group, with meetings and whatnot.

The purpose of their meeting was to work on a project that had been code-named CLOE (which I later looked up and apparently means Closed-Loop Output Error, referring to a certain class of algorithms.)  The goal of the CLOE project is to ingest logs about Prime ads shown and their result (converted or not), and when queried for a specific customer, consult CLOE's model and hand back the ad that would most likely result in a new conversion.

# Online CLOE and offline CLOE

I had a pretty good explanation of the two flavours: online CLOE and offline CLOE.  The *offline version* simply works off of existing Prime advertisement logs joined with customer data, and is potentially a little more transparent and easier to understand.

The *online version* learns on-the-fly as more information comes in from impressions, but its model is not quite as human-accessible, and is slightly more complex.  I discovered that if I were to choose this project, it would most likely be a variation of the offline version, where the end-goal becomes discovering the most likely predictors of ad success.  Cool!

After the meeting, I got some more explanations from Chris back near our desks (which I really don't need sometimes when my brain is fried!), and I spent some more time trying to get my Arch chroot working.  I pretty much decided it was impossible to get it functional due to the kernel being so old on my desktop machine and their sources being statically compiled against newer things, so I started trying to get a Gentoo chroot working instead.  I may abandon the project, write more on the progress later.

# Chose your destiny!

I also had my one-on-one meeting with Mark that day, in which Mark gave me the options of my intern project, and in which I chose one.  One option was doing a lot of UI improvement Prime's content-editing tool, making it so that editing templates wasn't so painful for marketers.  It would be a bunch of HTML, AJAX, working with WYSIWYG editors, etc.

The other option was the offline CLOE project---taking Prime customer impression data, and trying to discover variables that predict high success of specific ads.  Basically, since I have always done web development stuff, and I wanted to try something completely new, I chose the CLOE project.  I'll get experience working with clusters, processing huge amounts of data, and some machine learning.  It will be difficult, but awesome.

Afterwards, Chris was determined to get some environments deployed to my computer before my day-long tutorial tomorrow.  I tried to make it clear that I needed to leave early at around 4:30 PM today, since I wanted to see Monica and Mariana off on their bus, and he was actually quite accommodating in being aware of the time.

# Seeing the guests off (and busses)

So, just after 4:30 PM, I headed home as planned, hung out with the girls a bit more, heard their tales of visiting the [Pacific Science Center](http://www.pacificsciencecenter.org/) [[Yelp](http://www.yelp.ca/biz/pacific-science-center-seattle)], and then took them to their bus stop a two-minute walk north of my apartment, just past Denny Way at a Best Western.  This will be good to know if I wish to visit home.  A bunch of people started arriving, the bus came, and everyone got on.  Goodbye you two, thanks for visiting!

(Since writing there are now two bus companies that travel between Vancouver and Seattle: [Quick Shuttle](http://www.quickcoach.com/) and [BoltBus](https://www.boltbus.com/), the latter of which claims to have as low as $1 fares.  Very convenient for a weekend visit up to Vancouver!)

Afterwards, I came back home, sat around on my computer for the rest of the night, and didn't get anything particularly useful accomplished.
