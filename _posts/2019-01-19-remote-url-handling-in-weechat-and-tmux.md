---
layout: post
title: Remote URL handling in weechat and tmux
---

My approach to IRC (and a number of other chat systems, like Discord and Stack Exchange) is to run [WeeChat](https://weechat.org/) on my server inside [tmux](https://github.com/tmux/tmux). This is very convenient for me; I don't need to configure multiple chat clients, each with their own UI paradigms and credentials, on every machine I use, I just need to ssh home and `tmux attach`.

Where this _isn't_ convenient, though, is handling of links and especially images. Services like Discord and Slack support inline images, which get turned into links in IRC. I could manually copy-paste the links, but that's a pain. What if I could get weechat to intelligently open or preview them for me?

At first I looked into using [SIXEL](https://github.com/saitoha/libsixel), a format for inline graphics in the terminal, but most terminals don't support it, and even when I'm using one that does, I'd need special builds of weechat and tmux to handle them properly.

In the end, I opted for a simpler approach: if I'm ssh'd in from a machine that the server can ssh back into, opening a URL opens it in Chrome or an image viewer on that machine. If not, it splits the tmux view and opens it side-by-side with weechat in a terminal-based browser or image viewer.

## Keyboard shortcuts in weechat

The first thing I did was look for a weechat script that made URL handling easier, since if I had to manually copy-paste URLs anyways I might as well just paste them into the browser. I ended up settling on [url_hint.py](https://weechat.org/scripts/source/url_hint.py.html/), which has the virtue of not just doing what I want but also being well documented.

First, we install it and set up a convenient alias for it:

    /script install url_hint.py
    /alias add open_url /url_hint_replace /exec -bg weechat-open-url {url$1}

Now, any URL that appears will be annotated with a small superscript number, and entering `/open_url <number>` will open that URL. That's a bit awkward, though, so let's add some keyboard shortcuts...

    /key bind meta-o1 /open_url 1
    ...
    /key bind meta-o9 /open_url 9
    /save

And now pressing `alt-o <N>` will open the N'th URL using the /open_url alias, which itself runs the `weechat-open-url` command. Of course, `weechat-open-url` doesn't exist yet, so right now this does nothing.

The rest of the work happens inside the `weechat-open-url` script; we're done setting up weechat itself.

## Basic URL handling

Off we go to create the script that weechat is trying to call...

    $ touch ~/bin/weechat-open-url
    $ chmod a+x ~/bin/weechat-open-url
    $ nano ~/bin/weechat-open-url

The first draft of this is really simple: split the tmux view, open a text browser, display the URL. I'm writing this in zsh since it's what I'm used to, but it should work with minimal -- perhaps no -- modification in bash.

    #/usr/bin/env zsh

    tmux split-window -h -c ~ zsh -c "elinks -anonymous 1 '$1'"

The first `-c ~` is an argument to tmux itself, telling it to start in our home directory; the `zsh` and everything else after it is the command to run. We use `elinks -anonymous` so it doesn't stomp on any existing elinks sessions. Replace with lynx or w3m if you prefer those.

This is fine if we just want to read blog posts or something, but it doesn't really work for images. Let's see if we can improve on it.

## Image previews in the terminal

There are a lot of terminal image viewers out there. If your terminal supports SIXEL images, you can use `img2sixel`. Mine doesn't, so I opted to go with [timg](https://github.com/hzeller/timg). So now we have:

    #/usr/bin/env zsh

    case "$(echo "$1" | tr A-Z a-z)" in
      *.jpg|*.jpeg|*.png|*.gif|*:large)
        tmux split-window -h -c ~ zsh -c "timg '$1'"
        ;;
      *)
        tmux split-window -h -c ~ zsh -c "elinks -anonymous 1 '$1'"
        ;;
    esac

Unfortunately, this doesn't work, because `timg` doesn't support loading images from URLs. Fortunately, I'd already written a wrapper for it to handle that, as well as improving the UI somewhat:

    # in ~/.zshaliases

    function timg {
      for img in "$@"; do
        printf '\x1B[1m%s\x1B[0m\n' "$img"
        if [[ $img == http* ]]; then
          curl "$img" > /tmp/$$.img
          img="/tmp/$$.img"
        fi
        command timg -g ${COLUMNS}x$((2*LINES-4)) -U "$img"
      done | sed -E '$ d' | less -ReS
    }

This lets you feed it any number of image files and/or URLs, and it'll display them all in a pager, each one prefixed with the filename or URL in bold. With this defined, we update weechat-open-url accordingly:

    tmux split-window -h -c ~ zsh -c "source .zshaliases && timg '$1'"

And now, any URL in weechat can be opened in two keystrokes, to display in the terminal alongside weechat. Image previews are pretty low-res, but it's generally enough to get the gist of the image and let you know if you want to go to the effort of opening it "properly".

This is good, but we can do better.

## Remotely opening URLs on the client machine

Most of the time, I'm not connected from some random machine -- I'm at home, connected from my laptop, thoth. And the server can ssh into the laptop:

    ancilla:~ $ ssh thoth
    Last login: Sun Jan 13 21:59:53 2019 from 192.168.86.34
    thoth:~ $

So it seems like we should be able to use that to open URLs in the browser that's already open on the laptop; we just need a way of detecting where we're connected from. Fortunately, we can do that! First, we need to figure out which tmux client -- since there might be several connected -- I'm actually using.

    client_pid=$(tmux lsc -F '#{client_activity} #{client_pid}' | sort -g | tail -n1 | cut -d' ' -f2)

This uses `tmux list-clients` to list all connected clients and sort them by most recent activity; the one we're using _right now_ should be the most recently active. And once we have the PID of the client, we can figure out where we SSH'd in from, since sshd helpfully sets the SSH_CLIENT environment variable when you connect:

    client_ip=$(cat /proc/$client_pid/environ | tr '\0' '\n' | grep '^SSH_CLIENT' | cut -d= -f2 | cut -d' ' -f1)

Knowing the client IP, we're now good to go:

    function xssh {
      local host="$1"; shift
      ssh "$host" env DISPLAY=:1 \
        'XAUTHORITY=$(echo /run/user/$(id -u)/xauth_*)' \
        'XAUTHLOCALHOSTNAME=$(hostname)' \
        $@
    }

    function open-image {
      if [[ $client_ip == 192.168.86.101 ]]; then
        xssh "$client_ip" feh -x -F --auto-zoom "'$1'"
      else
        tmux split-window -h -c ~ zsh -c "source .zshaliases && timg '$1'"
      fi
    }

    function open-url {
      if [[ $client_ip == 192.168.86.101 ]]; then
        xssh "$client_ip" google-chrome "'$1'"
      else
        tmux split-window -h -c ~ zsh -c "elinks -anonymous 1 '$1'"
      fi
    }

    case $(echo "$1" | tr A-Z a-z) in
      *.jpg|*.jpeg|*.png|*.gif|*.svg|*:large)
        open-image "$1"
        ;;
      *)
        open-url "$1"
        ;;
    esac

Now, if we're connected from `192.168.86.101` (thoth's IP), it'll try to ssh back into that host and open images in `feh` and other URLs in Chrome. For everything else, it'll still display in the terminal.

In the future, I hope to investigate ways to open URLs on the client machine that don't rely on being able to ssh into it -- perhaps something involving reverse SSH tunnels -- but for now, this is fine.
