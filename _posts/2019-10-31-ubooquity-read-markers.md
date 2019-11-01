---
layout: post
title: Read markers in Ubooquity with JS and nginx
---

[Ubooquity](https://vaemendis.net/ubooquity/) is a web-based comic and ebook reader; I've never tested the ebook support, since I have an e-ink reader for that, but the comic support works well, and I run a local instance. However, the whole thing is pretty barebones, and missing features common in other readers like Perfect Viewer. The most aggravating lack, I found, was the lack of read-markers; it remembers where you are in each book, but doesn't tell you (in the book selection window) which ones you have and haven't finished.

The author has said that they plan to add this feature, but it's been nearly a year since the last release and there's no ETA for when it'll show up; and Ubooquity is closed-source, so I can't just hack the feature in and recompile.

Fortunately, there's another way: using JavaScript to query the server's memory of what page you're on.

Ubooquity has two ways of remembering where you are, depending on settings. If "store in cookies" is set, read progress is stored client side, in a `comic1234` cookie; the numbers are the server-side comic ID and the value is which page you were on last. I don't use this (since I read on a variety of devices) and haven't attempted to support it, although it wouldn't be hard to add such support.

The other approach stores them server-side, which lets them sync across devices and (if you have it configured to use user accounts) remember them per-user even on shared devices. Obviously, we can't access this information directly, but Ubooquity exposes an HTTP API for getting both the total number of pages (used for the "comic details" popup) and which page you were on last (used when actually opening a book).

(The code presented here is incomplete; the complete script is [here](https://raw.githubusercontent.com/ToxicFrog/misc/master/ubreader.js). I've added UserScript headers to it, but they are untested.)

We start out by figuring out what the baseURL is. Ubooq isn't always served from /; it might be served from, say, `www.example.com/ubooquity/` or the like, and we need to take that into account when making our requests.

    let baseURL = window.location.pathname.match("(/.*)/comics/[0-9]+")[1];

All the grid-of-thumbnails pages Ubooq uses use the `cell` class for the divs containing the thumbnails. All of them contain an `<a>`, but the `<a>` will only have an `onclick` if they're thumbnails for actual books; directory thumbnails just have an `href`.

    function updateAllReadStatus(_) {
      let cells = filter(
        document.getElementsByClassName("cell"),
        cell => { return !!cell.getElementsByTagName("a")[0].onclick; });
      for (let cell of cells) {
        let img = cell.getElementsByTagName("img")[0];
        let id = img.src.match("/comics/([0-9]+)/")[1];
        updateReadStatus(baseURL, cell, id);
      }
    }

The bookmark information can be fetched (in JSON format) at `/user-api/bookmark?docId=1234`; we're interested in the `mark` field. If it returns something other than 200 (in particular, if you've never opened that book it returns HTTP 204 No Content), we manually emit a bookmark on page -1, which will be handled later.

    function updateReadStatus(baseURL, cell, id) {
      fetch(baseURL + "/user-api/bookmark?docId=" + id)
      .then(response => {
        if (response.status != 200) {
          return Promise.resolve({"mark": "-1"});
        }
        return response.json();
      })

Of course[?], `mark` is a string, not an int, despite storing a page number; and it's 0-indexed, so if you're on page 1 of a 100-page book, it's 0, and if you've finished the book, it's 99. (There's also a separate `isFinished` boolean field, which is always false.)

    .then(json => {
      cell.bookmark = parseInt(json.mark) + 1;
      return fetch(baseURL + "/comicdetails/" + id);
    }).then(response => {
      return response.text();
    })

`/comicdetails/1234` contains the HTML fragment used for the "book information" popup that gives you options to "read" or "download". For some reason, the link to "read" includes the total number of pages in the query-string, so we fish that out, and if the number of pages read doesn't match the total, we add an "in progress" bubble to the thumbnail, showing the current page, page count, and an open book icon. If the number of pages read is 0 (from the `Promise.resolve` earlier), we add a closed book "not started yet" icon.

      .then(text => {
        let pages = parseInt(text.match("nbPages=([0-9]+)")[1]);
        if (cell.bookmark <= 0) {
          addBubble(cell, "📕");
        } else if (pages != cell.bookmark) {
          addBubble(cell, cell.bookmark + "/" + pages + " 📖");
        }
        fixupLinks(cell, text);
      })

Conveniently, Ubooq already has CSS set up to display an bubble overlaid on thumbnails; it's only used for directories, to show the number of files inside, but we can easily repurpose it here to display the unread/in-progress bubbles by constructing divs and spans with the appropriate classes. (The `fixupLinks` function is used to remove the "book details" interstitial.)

    function addBubble(cell, text) {
      let div = document.createElement('div');
      div.className = "numberblock";
      div.innerHTML =
        '<div class="number read-marker"><span>' + text + '</span></div>';
      cell.append(div);
    }

So there we go! Now the only problem is injecting it into the page. If you're using Greasemonkey or something like it, and are ok manually adding it as userJS to each device you read on, you can just do that; the version on github linked above even has UserScript headers to facilitate this. But I don't want to do that; I read on a bunch of different devices, with different browsers, and would rather have the code injected server-side. And since I have Ubooquity behind an nginx reverse proxy, I can do just that! Off to `configuration.nix` we go...

    virtualHosts."comics.example.com" = {
      locations."/".proxyPass = "http://127.0.0.1:2202";
      locations."/admin".proxyPass = "http://127.0.0.1:2203";
      locations."= /ubreader.js".alias = pkgs.copyPathToStore /path/to/ubreader.js;
      locations."/comics".extraConfig = ''
        sub_filter '</head>' '<script type="text/javascript" src="/ubreader.js"></script></head>';
        sub_filter_last_modified on;
        sub_filter_once on;
      '';
    };

Then we just drop `ubreader.js` in `/srv/ubooquity` (or wherever else is convenient) and away it goes! The script will be injected into every new page load, no matter what browser you access it from.

Happy reading!
