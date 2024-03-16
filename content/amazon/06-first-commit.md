---
title: "Amazon intern day 6: my first commit"
date: 2011-05-05
---

According to some memo that went out, today was Classy Thursday!  (Since Classy Thursday gives new meaning to Casual Friday.)  Apparently usually only this other Prime team participates in such an event, but I figured I may as well make an attempt.  So off to work I went in my classy shirt and my classy tie, and my questionably classy maroon pants and my even more questionably classy old blue Adidas.

I came across this Indian fellow in the elevator again (not the one from Amazon though---this one I met a day or two ago in the elevator, and he works somewhere downtown).  Most of our conversation consisted of talking about weather, which wasn't terribly long considering after leaving the building we didn't even go half a block before splitting up.

## Today's accomplishments

Work today was a lot of listening to long, drawn-out explanations from Chris; fighting with [synergy](http://synergy-foss.org/) (I still haven't figured out why, but the process completely freezes keyboard/mouse input in certain scenarios, forcing me to SSH into my workstation and kill the process [I later discovered this could be fixed by using a particular version on both the client and server]); and submitting my first commit through one of the tutorials on SDE Bootcamp (basically adding a line to a "guestbook" text file hosted on some webserver).  I am still having a tough time making the distinction between Amazon's two main packaging systems, but hopefully it will become clearer as time goes on.

## Lunch with a fellow intern

I decided to contact the other UBC intern (Peter) whom I met on orientation day to see if he wanted to go for lunch.  Apparently he already had brought one, but we met in the cafeteria anyway, and I got an enormous salad (the limit is four toppings, but I swear the salad man put about 32 on there!).  We chatted about our jobs, our housing, Vancouver, etc.  Then, another intern, Joshua, appeared, apparently also in the midst of eating lunch.  He sat down with us and I discovered he works in the same building as me.  I unfortunately had to go right then because I had a meeting (or at least thought I did), but I did exchange phone numbers with him.

## Defending your honour against the innocuous interview question

At the end of the day, a co-worker called Richard, Chris and I spent a long time discussing some interview questions.  (This turns out to be one of the favourite ways to derail a fellow employee from getting his or her work done for a good part of the day.)  The discussion was set off because I had thought up an answer to one Richard had mentioned at my welcoming pub night.  The question was this:

> Given `k` lists of `n` sorted numbers each, how would you produce a sorted list with all `n × k` numbers in it?

Here is my solution:

1. Initialize a priority queue.  There should be `k` elements---one for each list.  The key is the largest element from the list, and the value is a pointer to the list.  This could be implemented as a min-heap, which would allow for `O(lg n)` insertions and deletions.
2. Repeat until all lists are empty:
  1. Take the minimum key from the heap, and put the number in the sorted output list, taking note of the list from which it came.
  2. Next, to ensure the priority queue has a number from each of the `k` lists, take the next-smallest number from this particular list (provided one still exists), and put it in the priority queue along with its list pointer.

Thus, since there are `n × k` numbers in total, it is an `O(n × k × log(k))` algorithm.  After talking it over with Richard, this is apparently one of the two main solutions of the problem.  (The other is a more algorithmic approach, essentially accomplishing the same steps.)

# Intern dinner

Today, there was some emailing back and forth on the intern mailing list about the interns getting together this weekend for a dinner.  I think I managed to get people interested in doing something Friday night, which is good since I will have a visitor on Saturday and Sunday.  Hopefully it works out and I get to hang out with some other interns!

# First Thursdays

I ran off to the [Seattle Art Museum](http://www.seattleartmuseum.org/) (often called SAM), since it is free on the first Thursday of the month.  The museum wasn't quite as impressive as I'd hoped, but I did really enjoy some of the modern art there.  Also, for whatever reason, perhaps because of the current exhibits or the free Thursday event, there was a strange DJ playing ridiculously loud music from their lobby, which essentially reverberated throughout their entire two upper levels of exhibits, ruining the regular quiet museum feeling.  Also, their main exhibit involved people dressing in weird outfits with lots of fuzzy and strange-looking clothing.  Seattle, you are odd.

Later on, I would discover just how well-known and widely-celebrated [First Thursdays](http://www.firstthursdayseattle.com/) are.  It is an must to check out while in Seattle over the summer.  A few museums open up for free, and there is an awesome art walk that goes down in the [Pioneer Square](http://maps.google.ca/maps?q=pioneer+square+seattle&hq=&hnear=Pioneer+Square,+Seattle,+King,+Washington,+United+States&ll=47.608015,-122.333193&spn=0.044037,0.033989&t=m&z=14&vpsrc=0&iwloc=A) area of downtown Seattle.  My absolute favourite was the [619 Western Ave Arts Building](http://www.firstthursdayseattle.com/profile.php?id=97), a five-story building full of open studios during the art walk.  Unfortunately, it says they are no longer able to participate---I am not sure why, but it is very sad!

There are also smaller art walks, such as the [Blitz Capitol Hill Art Walk](http://www.blitzcapitolhill.com/), but First Thursday's art walk is the grandaddy.

# Graebel housing option number two

After the SAM, I checked out the next possibility that Graebel/Aboda had offered me earlier on today.  It's literally a block and a half away from my current place, and it looks decent.  I might try to see if I could tour any units tomorrow morning before work---but I am still very strongly considering just staying in my current place, despite the extra cost.

# Cinco de Ignoro

Finally, I got back to my place, and decided to go check out the tail end of the Cinco de Mayo festivities occurring in the clubroom.  (I had seen posters up around the apartment building advertising the event.)  It was extremely bizarre---I got in there and there were two tables occupied, and people both vaguely said "hello" to me as I walked in.  So I asked if there was more food, and they muttered "yes" and one said "certainly, help yourself to some burritos."

I grabbed one, and then half-heartedly sat down at one of the tables.  Everyone there seemed to be deep in discussion, but you would think that they would at least say "hello" to me.  Nothing!  No "are you new
here," no "where are you from," just continuing their current discussion, and ignoring my existence.

So, I got up, walked over to the counter for something, and tried the next table.  Same thing!  They seemed very deep in discussion about education systems, which is perhaps one possibility as to why they might not have bothered welcoming the newcomer.  But I suspect they probably saw some kid who looked ten years younger than them and they decided not to act as though I was a regular human being.  Go figure, Seattle.
