---
layout: post
title: Easily opening URLs from tmux on the client machine using custom OSC sequences
---

I use [`weechat`](https://weechat.org) for IRC (and a bunch of other chat systems). It runs on a remote server, and I connect to it with `ssh -t tmux attach`, and all is well.

Often, people post URLs that I want to open. For a while I just manually copy-pasted URLs from weechat into the local browser, but that got tedious fast. So I went looking and found [`url_hint.py`](https://weechat.org/scripts/source/url_hint.py.html/) -- it annotates URLs in the chat with numbers (or other symbols you choose), and gives you a command that opens them, which you can bind to a key. So now I can do this:

    /alias add url_hint_replace /exec -bg weechat-open-url {url$1}
    /key bind meta-o1 /open_url 1
    ...
    /key bind meta-o9 /open_url 9

And when a URL appears with a little ³ next to it, say, I can press `alt-o`,`3` and it'll run `weechat-open-url`.

Of course, it runs this on the *server*. So something that opens the browser on the server doesn't help me at all -- it needs to open it on the machine I'm running `ssh` from. Initially, I approached this using a script that figured out what machine I was connected from, using `tmux lsc` to find the PID of the tmux client and then reading `/proc/*/environ` to get the `$SSH_CLIENT` variable from it:

    client_pid=$(
      tmux lsc -F '#{client_activity} #{client_pid}' \
      | sort -g | tail -n1 | cut -d' ' -f2)
    client_ip=$(
      cat /proc/$client_pid/environ \
      | tr '\0' '\n' \
      | grep '^SSH_CLIENT' | cut -d= -f2 | cut -d' ' -f1)

Upon which it could, say, `ssh $client_ip google-chrome "'$url'"`. (It's a bit more complicated than that because it needs to set up `$DISPLAY` and `$XAUTHORITY`, but that's the gist.)

The problem here, of course, is that the server needs to be able to ssh back into the machine I'm running the client on. Which works great when I'm using the laptop on the home network, but not in most other situations. Ideally, I'd be able to pass the URLs to open down the existing SSH connection rather than opening a new one.

Conveniently, I can use the same technique I used for [DoomRL in the browser with sound](http://toxicfrog.github.io/doomrl-sound/): emit a terminal control sequence that something listening on the client will understand, and respond to -- in this case by opening the browser rather than by playing a sound.

Of course, it's not quite as simple as that, on either the client or server side.

On the client side, we need something that listens for the new control sequence. Modifying the terminal is presumably the "right" way to do this, but that's hard. An easier way is to wrap the command being run with something that will listen on stdout. The tool I wrote for this is [published here](https://github.com/ToxicFrog/misc/blob/master/oscwrap). It's pretty simple -- run the command, hook into stdout, watch for `OSC`, and if it sees one that it recognizes, do the thing. It recognizes two sequences, both initiated with `OSC 451` (which as far as I know is otherwise unused) -- one to open images (using the minimalist `feh` image viewer), and one to open other URLs (using the browser). In both cases it emits a notification first, in case it takes a while to load.

It doesn't bother allocating a pty or anything, so some things will probably break when run in it, but everything I've tried so far works fine.

Testing it locally is easy enough: `oscwrap printf '\x1B]451;URL;toxicfrog.github.io\x07'`, for example. And testing it remotely using `ssh` is just as easy. So, my first cut at `weechat-open-url` looks something like this:

    function osc {
      printf '\x1B]%s\x07' "$*"
    }

    function open-image {
      osc "451;IMG;$1"
    }

    function open-url {
      osc "451;URL;$1"
    }

    # Code to decide whether to call open-image
    # or open-url below this point...

Unfortunately, it doesn't actually work, because *tmux* sees the OSC before it ever hits my local tty and, not recognizing it, throws it away. Fortunately, tmux has its own control sequence for passing through control sequences to its parent terminal. So now we need to detect if running inside tmux, and wrap things accordingly:

    function osc {
      if [[ $TMUX ]]; then
        printf '\x1BPtmux;\x1B\x1B]%s\x07\x1B\\' "$*"
      else
        printf '\x1B]%s\x07' "$*"
      fi
    }

And now it works in and out of tmux. But if I run it inside of weechat, it still doesn't work -- either the output is discarded (using `/exec -bg`) or output to the chat window *with control codes stripped*. I need a way to bypass weechat's command wrapping entirely and write directly to its stdout.

Conveniently, `/proc` lets us do this:

    function find-tty {
      local pid=$$
      while [[ $pid != 1 ]]; do
        cat /proc/$pid/stat | read pid comm stat ppid _
        if [[ $(readlink /proc/$pid/fd/1) != /dev/null ]]; then
          echo -n "$pid"
          return
        fi
        pid=$ppid
      done
      echo "Error: couldn't find parent process with valid stdout" >&2
      echo -n "self"
    }

    function osc {
      local pid=$(find-tty)
      if [[ $TMUX ]]; then
        printf '\x1BPtmux;\x1B\x1B]%s\x07\x1B\\' "$*" > /proc/$pid/fd/1
      else
        printf '\x1B]%s\x07' "$*" > /proc/$pid/fd/1
      fi
    }

It is, of course, important to be careful not to emit anything printable when doing this, or anything that changes the cursor position or otherwise affects the display, because that will confuse weechat's output. It also blithely assumes that the first thing it finds as it ascends the process tree with stdout connected to something other than `/dev/null` is what it wants to be writing to, and there's probably some setup where this breaks hilariously. But it works for me!

This does mean that I now need to run `oscwrap` on the client -- but if I do that, it works from anywhere, and -- since it only needs to pass the URL down the connection rather than open an entirely new SSH session -- it is much faster. And the `oscwrap` script is easily extended to support other codes, if I need them.
