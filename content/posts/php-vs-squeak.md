---
title: "PHP vs. Squeak: a breakup"
date: 2009-06-17
draft: false
---

*What follows is an open letter to PHP, with the reasons why I have decided to break up with it.*

PHP, there used to be a spark in between us.  Every time I would see you, my heart would light up, cogs would start whirring in my brain, and other stuff would happen too.

I tried to look past your strange naming schemes.  I tried to look past your tacked-on object-oriented support.  Other languages did come by and tempt me---Python, Perl, even MySQL at times---but I remained faithful, I swear.  At least, I tried to.

But it's too hard---I can't do a long-distance anymore.  I have found someone else to spend my days with.  Someone who understands me.  Where you were an unresponsive and inactive partner, Squeak patiently waits and helps us grow together as we go along.  When things go wrong, Squeak is waiting there to help fix things.

# Top 5 reasons to use Squeak over PHP

And so, here are the top 5 reasons why I have decided to break up with you, PHP:

1. Where you had ugly `print_r` statements and clunky [xdebug](http://xdebug.org/) interfaces, Squeak pipes up with its tested and proven **debugger with a full stack trace**.

2. Your function arguments are difficult to remember.  Certainly I can look them up on [php.net](http://php.net) or use an IDE to find out, but in Smalltalk the **arguments are built-in to the message name**.  For example: `httpPostDocument:args:`, versus: `http_post_document($var1, $var2)`.

3. When you're needed, you grudgingly grind into existence, loading all your extensions and libraries.  Squeak, like Python web applications, starts up once and **stays running**.  You can get fantastic speed increases this way.

4. **Squeak is portable**.  It runs anywhere, on any platform---offline development becomes a snap, as all you need is your image and a Squeak VM executable.  No need to set up Apache, PHP, deal with file permissions, etc.

5. And finally, **Squeak is alive**.  I'm not writing out dead `.php` files and hoping errors don't crop up when I run them.  Squeak has no concept of files, and instead constantly compiles code as you separate methods, correcting your syntax errors on-the-spot.  I can execute bits of code while I am writing them to test them out.  No need to open up a new console and run `php -r 'some code here'`.

# Recruitment

Hopefully you will understand why I have left you.  And although it might seem vindictive, I won't hesitate from convincing others to do the same.  If anyone out there is reading this, please look into something else.  Even if it isn't Squeak, do yourself a favour and borrow from Obama's platform: try [Django](http://django.com/) or [Seaside](http://seaside.net/), or any other web framework not built with PHP.
