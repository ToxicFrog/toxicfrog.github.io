---
layout: post
title: Improved read markers in Ubooquity
---

Last fall I [posted about](/ubooquity-read-markers/) a javascript injection for adding useful read markers to the Ubooquity comic server. I haven't been idle since then, since I kept tripping over areas for improvement while trying to read things, and I've finally gotten around to cleaning up and releasing those improvements:

- Fetching of the read markers is now delayed 1000ms from page load; this means you have to wait a moment before the markers appear (or before the book links are rewritten), but it works around an issue where sometimes, after closing a book, it takes the server a moment to update the read status, meaning that when the page loaded it would fetch the old status rather than the current one.
- A reload button is added to the top center of the bookshelf page
- The progress bar in reading mode is replaced with a draggable slider that lets you easily seek to any page; the slider, page counter, and actual reading of the book should always stay in sync.

The script is still available [on github](https://raw.githubusercontent.com/ToxicFrog/misc/master/ubreader.js) and as before it has (untested) UserJS markers if you want to try installing it in the browser rather than injecting it into the pages server-side.
