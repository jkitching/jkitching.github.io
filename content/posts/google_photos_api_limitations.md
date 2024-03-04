---
title: "Unreasonable limitations of Google Photos API"
date: 2024-03-04
draft: false
---

**Beware:** Google Photos API is *not* suitable for reorganizing your photo collection.  From [Google for Developers documentation](https://developers.google.com/photos/library/guides/manage-albums):

> `batchAddMediaItems`: Note that you can only add media items that have been uploaded by your application to albums that your application has created.
>
> `batchRemoveMediaItems`: Note that you can only remove media items that your application has added to an album or that have been created in an album as part of an upload.

In other words, unless you are fully managing the contents of your Google Photos account via the API (uploading photos, creating albums, adding photos to albums), Google Photos API is only useful for **reading**, and not for **writing**.

Issues on the Google public issue tracker ([b/109505022](https://issuetracker.google.com/issues/109505022), [b/132274769](https://issuetracker.google.com/issues/132274769)) date back to as early as 2018, so it seems unlikely to change anytime soon.

Thanks to the article [Google Photos API Giveth, and Then Promptly Taketh Away](https://www.blogbyben.com/2020/01/google-photos-api-giveth-and-then.html) for spelling this out.  
