---
layout: post
title: "Share By Link" with fgallery and nginx
---

For a while I've wanted a better solution to personal file sharing. Just tossing stuff into `/srv/www` works, but makes it hard to isolate files shared with one person from files shared with anyone else. Nor does it have a good UX for sharing photo galleries, or a convenient way to say "shove all these files into a zip and share that". Using Google Drive and Google Photos solves that, but has issues of its own, like making it easy to leak personal information and not having a line client.

So, I [wrote my own](https://github.com/toxicfrog/misc/blob/master/share).

Early versions of this were based on IPFS, but after some experimentation I concluded that IPFS was massive overkill for what I wanted to do, especially since, as I wanted to be able to update shares in place, I needed to add multiple layers of indirection (IPNS for a stable name, then IPFS for the actual share).

The current version just works by hashing the name of the share and the user ID (so multiple users using it won't collide) using `sha256`, postprocessing the output to make it URL-friendly, and then copying the contents of the share into `/srv/www/share/<hash>/`. Metadata is written to a `.share` file in that directory, and is not currently used except by `share ls`, but at some point will be used to support a `share refresh` command that automatically regenerates the share from the original input files/directories. Actual serving of the files is done by any static webserver, in my case `nginx`.

In addition to being able to create shares out of files and directories, it has convenience functions for creating zip files and photo galleries. The former just invokes `zip` on the inputs and is primarily intended for cases where you want to share an entire set of files as a unit, rather than letting users pick and choose what they download. The latter uses `fgallery` to generate a static javascript-based thumbnailed image gallery and `exiftool` to strip metadata from the images.

Since `fgallery` doesn't support GIFs or video files, the script also generates PNG previews for those and feeds them to `fgallery` instead, then postprocesses the generated gallery so that clicking through on the preview direct-links to the original GIF or video, which most browsers will display inline rather than downloading.

It has a bunch of dependencies -- in addition to the ones mentioned above, it also needs `xxd`, `pv`, `rsync`, `convert` (from ImageMagick), and `base64`, plus a few others -- but it should check for missing dependencies on startup and tell you what you're missing, and everything it depends on is fairly popular and should be available in your friendly local software repo.
