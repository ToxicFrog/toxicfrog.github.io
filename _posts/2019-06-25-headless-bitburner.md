---
layout: post
title: Headless BitBurner with VNC
---

I enjoy the occasional idle/incremental game. I generally prefer the relatively short ones with clearly defined goals and story arcs, like Spaceplan or Paperclips, but I do enjoy larger ones like Trimps or KittensGame on occasion (although I have finished neither of those, and probably never will). Of late, my poison of choice has been [BitBurner](https://danielyxie.github.io/bitburner/), a cyberpunk incremental that is less about playing the game directly and more about writing javascript (or anything else you care to write that compiles to JS) to play the game for you.

Since I enjoy programming and generally bail on games like this once they start introducing tedious mechanics that can't be automated away, I'm sure you can see why a game entirely designed around automating everything with programs might appeal to me.

The big issue with this (and most other games of its ilk) is that you have to keep it running. If you close the browser tab -- or in some cases just unfocus it! -- it switches to a slow and approximate "offline mode" that only gets you a fraction of whatever progress you would have gotten had you left it running. This is especially true in Bitburner, where it being online and executing JS is a prerequisite of your programs running at all, and there are major limits to what it can infer about what you "would have done" had it been left running.

This, of course, means I need a place to run it. The desktop spends a lot of time asleep and I'm not often using it. The laptop spends less time asleep, but leaving Bitburner (or Trimps or whatever) running noticeably impacts battery life. And the server is headless.

So, I decided to run it on the server.

Historically I've preferred `ssh -XC` (for stuff that doesn't need to stick around) or `nxplayer` (for stuff that does). NX has gotten pretty heavyweight since last time I used it, though, and the server isn't available in nixpkgs. So I turned to VNC. Historically, I've avoided VNC because remoting an entire desktop when I really just wanted a single program seemed like overkill, but it can, it turns out, be made fairly minimal.

I'm using TigerVNC for this because (a) it's available in nixpkgs and (b) it supports both `RemoteResize` and automatic SSH tunneling, but this setup should work with pretty much any VNC server and client, and other clients offer features that might, depending on your setup, be more attractive than SSH tunneling, like client-side scaling.

The server-side setup, it turns out, is extremely simple:

    $ mkdir -p ~/Games/Bitburner
    $ cd ~/Games/Bitburner
    $ nano bitburner.xinit
    #!/usr/bin/env bash

    ratpoison &
    exec google-chrome-stable \
      --user-data-dir=$HOME/Games/Bitburner \
      --app=https://danielyxie.github.io/bitburner/

    $ chmod a+x bitburner.xinit

We need some window manager to make sure Chrome behaves, so `ratpoison` is used since it has minimal footprint and does what we want (makes Chrome fullscreen all the time) with no configuration needed. Chrome is run in `--app` mode to hide the address bar and whatnot, although nothing stops you from opening new tabs with `^T` or the like.

Then you just start up the VNC server:

    vncpasswd
    vncserver -autokill -localhost \
      -xstartup ~/Games/Bitburner/bitburner.xinit

And connect to it:

    vncviewer -RemoteResize -via me@server -fullscreen localhost:1

The `via` is what sets up the SSH tunnel; if you're on the same LAN and comfortable using direct VNC connections you can drop the `-localhost` from the server and `-via` from the client and just use `vncviewer server:1`.

This is, of course, very easily adapted to run other idle games, or multiple idle games at once -- whether in different VNC sessions or different tabs in the same session.

One thing that I tried to get working, but could not, was `Xvnc`'s built in httpd. It's meant to be for serving the Java version of the VNC client (so people can VNC in from their browsers), but you can point it at any directory you wish. My plan was to point it at the directory I develop Bitburner scripts in so that I could easily fetch them into the game (since there's an in-game version of wget that lets you download scripts from the internet), rather than needing to copy-paste each one. However, it just served empty responses. Since I don't really care to spend forever debugging it, it's likely that I'll just use `python -m http.server` instead.

For completeness, here's the NixOS configuration I used for the server:

    { config, pkgs, lib, ... }:

    {
      # Bitburner uses a bunch of emoji and special characters.
      # I probably don't need *all* of these, but why not?
      fonts = {
        enableDefaultFonts = true;
        enableFontDir = true;
        enableGhostscriptFonts = true;
        fontconfig.cache32Bit = true;
        fonts = with pkgs; [
          corefonts
          google-fonts
          gentium
          inconsolata-lgc
          noto-fonts-emoji
          symbola
          unifont
          unifont_upper
        ];
      };

      # Since this is a user service, it's not enabled by default, and due to
      # a long standing bug in nixpkgs, you can't `systemctl --user enable` it
      # either. Instead, enable it by hand:
      #   mkdir -p ~/.config/systemd/user/default.target.wants
      #   ln -s /etc/systemd/user/bitburner.service \
      #        ~/.config/systemd/user/default.target.wants
      # and then `systemctl --user start bitburner`.
      systemd.user.services.bitburner = let
        # writeScript will write this to the nix store with appropriate shebang
        # and permissions and return its path.
        xinit = pkgs.writeScript "bitburner.xinit" ''
          ${pkgs.ratpoison}/bin/ratpoison &
          exec ${pkgs.google-chrome}/bin/google-chrome-stable \
            --user-data-dir=$HOME/Games/Bitburner \
            --app=https://danielyxie.github.io/bitburner/
        '';
      in {
        description = "Headless Bitburner session running in VNC";
        after = ["network-online.target" "local-fs.target"];
        preStart = "mkdir -p $HOME/Games/Bitburner";
        serviceConfig.ExecStart = "${pkgs.tigervnc}/bin/vncserver -fg -autokill -xstartup ${xinit} -localhost";
        path = with pkgs; [perl xorg.xauth];
      };
    }
