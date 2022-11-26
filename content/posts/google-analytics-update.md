---
title: "Google Analytics midterm update"
date: 2009-07-08
draft: false
---

I would like to give an update as to how the [Google Analytics API](http://www.drupal.org/project/google_analytics_api) project is going to date.  Here is how it has progressed since the start of the GSoC term:

# A library to reuse

I found an existing PHP library called [Google Analytics API PHP Interface (GAPI)](http://code.google.com/p/gapi-google-analytics-php-interface/) that had support to access the Google Analytics Data Export API, but it lacked AuthSub support, which is arguably one of the more flexible authentication methods to access Google Services.  Thus I spent time looking into the auth documentation, and hacked in support to the library.  I started tracking my changes into the [Drupal.org CVS repository](http://cvs.drupal.org/viewvc.py/drupal/contributions/modules/google_analytics_api/).

# Licensing issues

Unfortunately, I soon found out from my mentor the unfortunate news that the library's license (GPL3) was incompatible with that of Drupal's (GPL2+) and therefore was not allowed in the repository.  I contacted the owner of the GAPI library, Stig Manning, and luckily he was glad to let me merge my changes back into the library's repository on Google Code, and in addition he had no problem re-licensing it so that it could be packaged with the Drupal module.

# Implementing admin interface

At that point I started working on merging the changes back into the GAPI repository.  Late night hacking then resulted in the following solid features of the module:

- An administration page where you can log into and out of a Google Account via AuthSub.
- A testing query page where you can fill in your metrics and dimensions and see what data Analytics will send back.

# Future plans

According to the [wiki schedule](http://groups.drupal.org/node/22599), all that needs doing before midterms is starting to figure out a GUI. So here are the future plans, including after the midterm submission:

- Create a tab beside "View" and "Edit" on nodes (not sure how to do this yet) that allows you to view said statistics.
- Create a block that is small enough to fit in a sidebar.  Show some quick statistics about the node you are currently looking at.
- Views integration.  Decide on the statistics that could easily be used to filter pages, and include them in Views.
- And for later on in the project: A dashboard page that aggregates some overall interesting statistics of the site.

# Testing

I would love to start getting some people to test out the module and send feedback/flames/comments my way.  It's the first major Drupal module I have developed from scratch, so if you see things that are done a little strangely, feel free to point out a better way.  I haven't released any versions yet so you'll need to check it out via everyone's favourite source control program:

```sh
cvs -z6 -d:pserver:anonymous:anonymous@cvs.drupal.org:/cvs/drupal-contrib \
    checkout -d google_analytics_api-HEAD google_analytics_api/
```

Please take a look! It'll be most meaningful if you have a Google Analytics profile with a decent amount of pre-existing traffic on it.

But overall, so far so good. I am looking forward to the second half of this coding term. Thanks to everyone for sending emails with your interest in the project!
