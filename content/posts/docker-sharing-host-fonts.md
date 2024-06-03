---
title: "Sharing fonts from host to docker instance"
date: 2024-06-03
---
As part of writing [html_to_pdf.sh](https://github.com/jkitching/1pager-printable-html/blob/master/html_to_pdf.sh) for programatically printing a webpage to PDF format, I encountered the problem of fonts appearing differently after having been output as PDF.  Further investigation revealed the `alpine-chrome` Docker image did not have the fonts being used in the HTML document.

A simple solution---replace `/usr/share` font directories in the Docker image with those of the host:

```sh
docker run --rm \
    -v /usr/share/fonts:/usr/share/fonts \
    -v /usr/share/fontconfig:/usr/share/fontconfig \
    asia.gcr.io/zenika-hub/alpine-chrome
```

And it works!
