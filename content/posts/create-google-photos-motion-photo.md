---
title: "Convert iPhone Live Photo to Google Photos motion photo"
date: 2024-05-27
---

Google Photos has two methods of handling motion photos:

1. If the photo was uploaded by an Android device, it will already be in Google's custom format, which is essentially a JPEG file with a video file concatenated to the end.

2. If the photo was uploaded by an iPhone, it will be stored as two separate files on Google servers: HEIC and MOV.

When downloading motion photos and re-uploading them to a different Google Photos account, the Android case works fine, since everything is packed together as one file.  In the iPhone case, however, you will end up uploading the HEIC and MOV files separately, which will not be recognized by Google Photos as a motion photo, and will instead displayed as two separate entities.  What's the solution?

# Script for creating a motion photo

Download script from GitHub: [combine_photo_video.sh](https://gist.github.com/jkitching/3fa5a0c238fa825278db83acd05d7742)

This script takes a photo file and a video file, and combines the two into the format recognized by Google Photos.  It depends on [exiftool](https://exiftool.org/) for adding the XMP block.

Simply run the script with three arguments: 

```sh
./combine_photo_video.sh <photo_file> <video_file> <output_file>

# For example, combine IMG_9077.heic and IMG_9077.mov:
./combine_photo_video.sh IMG_9077.heic IMG_9077.mov IMG_9077_motion.jpg
```

Upload the resulting file to Google Photos (either through the app or web version), and you will get a motion photo!

# Technical explanation

Motion photo JPEG files have an extra XMP block.  There are [two different versions in use](https://linuxreviews.org/Google_Pixel_%22Motion_Photo%22).  The newer one has the advantage of assuming the client will locate the video file on its own, and appears to work even when specifying the video file length as "0":

```xml
<Container:Directory>
  <rdf:Seq>
    <rdf:li rdf:parseType="Resource">
      <Container:Item
        Item:Mime="image/jpeg"
        Item:Semantic="Primary"
        Item:Length="0"
        Item:Padding="0"/>
    </rdf:li>
    <rdf:li rdf:parseType="Resource">
      <Container:Item
        Item:Mime="video/mp4"
        Item:Semantic="MotionPhoto"
        Item:Length="0"
        Item:Padding="0"/>
    </rdf:li>
  </rdf:Seq>
</Container:Directory>
```

More about the actual code for parsing and extracting video data here: [shotwell issue 233](https://gitlab.gnome.org/GNOME/shotwell/-/issues/233#note_1712445).

# Limitations

* Unfortunately, HEIC files need to be converted to JPEG as part of this process.
* Existing XMP data is wiped out as part of this process.  But it is unlikely that it exists anyways, unless the image file has been edited with Adobe products.
