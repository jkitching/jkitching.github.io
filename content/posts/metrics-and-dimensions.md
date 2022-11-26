---
title: "Metrics and dimensions"
date: 2009-06-16
draft: false
---

I have never learned anything about web analytics before.  This year, my submission to Google Summer of Code for a Drupal module ([Google Analytics API](https://www.drupal.org/project/google_analytics_api)) that makes use of the recently-released [Google Analytics Data Export API](http://code.google.com/apis/analytics/docs/gdata/gdataDeveloperGuide.html) was accepted.  I have been doing a lot of reading about the vocabulary used, the methods of filtering data, etc.

One distinction that is crucial to understand in querying the Google servers for analytics data is the difference between *metrics* and *dimensions*.

**Metrics** are the actual hard pieces of tangible statistics themselves.  These can include, for example:

- the length of a visit
- total page views
- the number of clicks on ads to get to your site
- entrances and exits to your site

**Dimensions** are the ways one can limit the scope of the metrics returned.  Ways of doing this can include:

- a date range (start and end date)
- the address or URL of a page
- the browser used
- the resolution size
- the originating country

# In practice

Without dimensions, you get the overall statistic for the entirety of your site.  But when queried in conjunction with dimensions, you can get some interesting data.

For example: *Give the number of **page views** with respect to different **countries***.

Or, in Google parlance: *Select the **ga:pageviews** metric in conjunction with the **ga:country** dimension*, giving something that looks like this, which in actual fact looks much like an SQL query result:

|Country|Page Views|
|--|--|
|Algeria|14|
|Belgium|39|
|Canada|49|
|...||

The possibilities are boundless (okay, they are bounded by dimensions), and I am sure there will be many practical applications in Drupal.
