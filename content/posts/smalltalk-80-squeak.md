---
title: "Smalltalk-80 and Squeak: an introduction"
date: 2009-05-29
draft: false
---

Riddle me this: given the success of the Java programming language and object-oriented programming in general, why has no one heard of Smalltalk?  Smalltalk is the first fully object-oriented language, older than the great K&R C itself.

# Dabble DB takes my Smalltalk virginity

In January 2009, through my university, I started a co-op work term with a company called Smallthought Systems.  They're the guys behind the award-winning web app [Dabble DB](http://dabbledb.com/).

On the job listing it noted applicants would be programming in Smalltalk, which I had never heard of at the time.  Here's a conversation we had on my first day:

> Them: Ever programmed in Smalltalk?
>
> Me: Nope.
>
> Them: Well, trust us, once you''ve tried it there''s no going back.  Or if you do have to program with something else, you''ll hate every minute of it.

I laughed.  Of course nothing that was decades old could be so radically ground-breaking.  Boy was I wrong.

# And you call this programming?

I'm not going to lie---Squeak is weird.  With Squeak, you don't open up your text editor, type out some code, hit "save", compile said file, and run the binary.

You use the Squeak Virtual Machine to open up an image---a saved state of an entire implementation of Smalltalk---wherein everything is programmed in Squeak.  The graphical interface is programmed in Squeak.  The compiler is programmed in Squeak.  And everything is modifiable.

# Some history

Those images themselves are actually descendants of the one used by researchers at XEROX PARC.  Nowadays, there are a number of different implementations forked from the original image, including a free and open-source one called [Squeak](http://www.squeak.org/) (if you listen to [FLOSS Weekly](https://twit.tv/shows/floss-weekly) you'll hear about it in [Episode 29](https://twit.tv/shows/floss-weekly/episodes/29), in which the Smalltalk and Squeak co-creator Dan Ingalls is interviewed).
