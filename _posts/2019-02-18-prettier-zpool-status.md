---
layout: post
title: A prettier zpool status
---

I run ZFS on my server, and that means I'm often looking at the output of `zpool status`, especially when shuffling drives around. Out of the box, however, it's kind of bland and has lots of wasted space:

![zpool status](/images/zpool-status-plain.png)

Using the [subcommand wrapper](/subcommandifying/) from the last post, though, we can easily replace this with something prettier. I'll be using terminal control codes for this; a full discussion of them is outside the scope of this post, but the only one I'll be using is `ESC [ ... m` anyways, which controls the styling of text. (If you want to know more, the [XTerm Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) page is extensive, and the sequences described there are supported by most terminal emulators.)

For starters, let's just make the name of the pool bold:

    function zpool-status {
      command zpool status "$@" | sed -E '
        /^  pool:/ s/pool: (.*)/pool: \x1B[1m\1\x1B[0m/
      '
    }

That sed script is going to get a lot larger, though. Next, let's compress things vertically by removing all the blank lines except the one at the end of each status block, as well as the errors: line if there are no errors to report:

    /^$/ d;
    /errors: No known data errors/ s,.*,,
    /errors: .*/ s,$,\n,

This replaces the "no errors" line with the blank line we want, and simply inserts a newline at the end of other error lines.

Now, we move the "config:" prefix to the same line as the table header, and underline the header itself (by deleting the config: line entirely and inserting that prefix at the start of the next line):

    /^config:/ d
    /^\tNAME/ { s/^./config: /; s/NAME.*/\x1B[4m\0\x1B[0m/; }

We can also replace the `ONLINE ... (resilvering)` status with a new `RESILV` status:

    s,ONLINE(.*)  \(resilvering\),RESILV\1,

And with that sorted out, make things more colourful! ONLINE will be green, RESILV blue, DEGRADED yellow, and OFFLINE, FAULTED, REMOVED, and UNAVAIL red:

    s,ONLINE,\x1B[1;32m\0\x1B[0m,
    s,RESILV,\x1B[1;36m\0\x1B[0m,
    s,DEGRADED,\x1B[1;33m\0\x1B[0m,
    s,(OFFLINE|FAULTED|REMOVED|UNAVAIL),\x1B[1;31m\1\x1B[0m,g

And finally, we can hilight non-zero error counts in red, too. For the sake of simplicity this code makes the assumption that only those lines end with three space-separated numbers, and that the device names on the same line will never contain whitespace followed by digits.

    / [0-9]+ +[0-9]+ +[0-9]+$/ {
      s, ([1-9][0-9]*), \x1B[1;31m\1\x1B[0m,g
    }

And there we go!

![zpool status](/images/zpool-status-pretty.png)

The whole thing put together can now be put in `~/.zshrc`, and as long as `enable-subcommands-for zpool` is run at some point, it'll just work.

    function zpool-status {
      command zpool status "$@" | sed -E '
        # name of pool in bold
        /^  pool:/ s/pool: (.*)/pool: \x1B[1m\1\x1B[0m/
        # delete blank lines
        /^$/ d
        # if no errors, replace "errors:" line with blank line; otherwise preserve
        # and append newline
        /errors: No known data errors/ s,.*,,
        /errors: .*/ s,$,\n,
        # move config: prefix to the start of the table header; underline the header
        /^config:/ d
        /^\tNAME/ { s/^./config: /; s/NAME.*/\x1B[4m\0\x1B[0m/; }
        # If any of the error counts are non-zero, hilight them in red
        / [0-9]+ +[0-9]+ +[0-9]+$/ {
          s, ([1-9][0-9]*), \x1B[1;31m\1\x1B[0m,g
        }
        # remove (resilvering) and replace the status with it
        s,ONLINE(.*)  \(resilvering\),RESILV\1,
        # make ONLINE green, RESILV blue, DEGRADED yellow, and other statuses red.
        s,ONLINE,\x1B[1;32m\0\x1B[0m,
        s,RESILV,\x1B[1;36m\0\x1B[0m,
        s,DEGRADED,\x1B[1;33m\0\x1B[0m,
        s,(OFFLINE|FAULTED|REMOVED|UNAVAIL),\x1B[1;31m\1\x1B[0m,g
      '
    }
