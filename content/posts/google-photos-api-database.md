---
title: "Creating a JSON database of photos and albums using Google Photos API"
date: 2024-03-12
---

It turns out that using the Google Photos API is [not particularly suited]({{< ref "google-photos-api-limitations" >}}) to organizing your photos collection.  On the other hand, the API makes it easy to create a listing of all photos and albums in your collection.

For the case of creating a simple file-based JSON database, I wanted a command-line driven solution, so I used [photoslibrary1-cli](https://github.com/Byron/google-apis-rs/tree/main/gen/photoslibrary1-cli).

# Authentication

The photoslibrary1-cli [documentation](https://byron.github.io/google-apis-rs/google_photoslibrary1_cli/index.html) claims to include a client secrets file for public use, but it no longer works.

Instead, [you will need to create your own](https://support.google.com/cloud/answer/6158849):
1. Set up a project on Google Cloud Console
2. Enable the Google Photos API
3. Choose the type of credentials to create (I used OAuth 2.0)
4. Download a JSON client secrets file

Then, copy your client secrets file into `./google-service-cli/photoslibrary1-secret.json`, and authenticate with the browser (if using OAuth 2.0) by running any photoslibrary1-cli command, e.g. `photoslibrary1 media-items list`:
> Please direct your browser to [URL] and follow the instructions displayed there.

# Pagination

photoslibrary1-cli does not handle pagination, so I wrote [photoslibrary1\_paginated.sh](https://gist.github.com/jkitching/236f62745c00a8d0578759e12b3f4502) to do the extra bookkeeping.

It takes four arguments:

1. `--target-array`: The property name which stores the list of results for the given request.  `mediaItems`, `albums`, or `sharedAlbums`.
2. `--command`: The photoslibrary1-cli command to run.  These are simply arguments forwarded directly to `photoslibrary1`.
3. `--output-name`: The name of the file to which items will be concatenated is `<output-name>.json`.
4. `--page-token-arg`: I believe this may be a quirk of the way photoslibrary1-cli was written.  Sometimes page tokens are provided as `-p page-token=<page-token>` and sometimes as `-r page-token=<page-token>`.

Page token is saved as `<output-name>.token`.  The script can be safely interrupted with Ctrl+C.  Just re-run with the same `--output-name` to resume.

# JSON database structure

This is how we plan to structure our database:

```sh
./media-items.json
./albums.json
./albums/ALBUM_ID.json
./shared-albums.json
./shared-albums/ALBUM_ID.json
./media-item-to-albums/MEDIA_ITEM_ID.json
./media-item-to-shared-albums/MEDIA_ITEM_ID.json
```

Each file contains one JSON object per line.  Let's build up the database step-by-step.

# Retrieve a list of media items

Save information about each media item (photo or video) into `media-items.json`.

```sh
./photoslibrary1_paginated.sh \
  --target-array mediaItems \
  --command 'media-items list' \
  --output-name media-items \
  --page-token-arg p
```

# Retrieve a list of albums

Albums that **you created** are listed here, even if they are shared with others.  Save to `albums.json`.

```sh
./photoslibrary1_paginated.sh \
  --target-array albums \
  --command 'albums list' \
  --output-name albums \
  --page-token-arg p
```

# Retrieve a list of shared albums

Albums that **others shared with you** are listed here.  Save to `shared-albums.json`.

```sh
./photoslibrary1_paginated.sh \
  --target-array sharedAlbums \
  --command 'shared-albums list' \
  --output-name shared-albums \
  --page-token-arg p
```

# Retrieve a list of media items in each album

For each album, retrieve a list of media items.  Save to `albums/ALBUM_ID.json`.

```sh
mkdir albums
for album in `cat albums.json | jq -r '.id'`; do
  echo "$album"
  ./photoslibrary1_paginated.sh \
    --target-array mediaItems \
    --command "media-items search -r . album-id=$album" \
    --output-name "albums/$album" --page-token-arg r
done
```

# Retrieve a list of media items in each shared album

For each shared album, retrieve a list of media items.  Save to `shared-albums/ALBUM_ID.json`.

```sh
mkdir shared-albums
for album in `cat shared-albums.json | jq -r '.id'`; do
  echo "$album"
  ./photoslibrary1_paginated.sh \
    --target-array mediaItems \
    --command "media-items search -r . album-id=$album" \
    --output-name "shared-albums/$album" --page-token-arg r
done
```

# Map media items to albums containing them

For each media item, if one or more albums contain the media item, create a file listing those albums.  Save to `media-item-to-albums/MEDIA_ITEM_ID.json`.

```sh
mkdir media-item-to-albums
for fname in albums/*.json; do
  album_id=`echo $fname | sed -e 's@\.json$@@' -e 's@^albums/@@'`
  echo "$album_id"
  for media_id in `cat $fname | jq -r '.id'`; do
    touch "media-item-to-albums/${media_id}.json"
    grep "$album_id" albums.json >> "media-item-to-albums/${media_id}.json"
  done
done
```

# Map media items to shared albums containing them

For each media item, if one or more shared albums contain the media item, create a file listing those albums.  Save to `media-item-to-shared-albums/MEDIA_ITEM_ID.json`.

```sh
mkdir media-item-to-shared-albums
for fname in shared-albums/*.json; do
  album_id=`echo $fname | sed -e 's@\.json$@@' -e 's@^shared-albums/@@'`
  echo "$album_id"
  for media_id in `cat $fname | jq -r '.id'`; do
    touch "media-item-to-shared-albums/${media_id}.json"
    grep "$album_id" shared-albums.json >> "media-item-to-shared-albums/${media_id}.json"
  done
done
```

And that's all!
