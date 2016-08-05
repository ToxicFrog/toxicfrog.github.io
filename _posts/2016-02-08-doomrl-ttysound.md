---
layout: post
title: Closed captions for DoomRL
---

[doomrl-server](https://github.com/ToxicFrog/doomrl-server) now supports closed captions, via a library that replaces libSDL_mixer. These are enabled by default on the [ancilla DoomRL server](http://phobos.ancilla.ca/).

If you want to use them locally, [the ttysound README](https://github.com/ToxicFrog/doomrl-server/tree/master/ttysound) explains how; it can be used both in tty mode (in which case the captions appear on an extra line at the bottom) and in tiled mode (in which case they appear in the window's title bar -- I'd like to add them to the HUD, but I haven't figured out how and that may not even be possible).

At some point I may even get around to packaging a prebuilt, "unpack and play" release for windows and linux.
