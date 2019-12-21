---
layout: post
title: DoomRL in the browser -- with sound!
---

The [public DoomRL server](https://phobos.ancilla.ca/) I host now supports sound and music, as long as you're playing it in the browser (as opposed to connecting via telnet). If you don't care about the technical details and just want to play, you can stop reading now -- sound is enabled by default for browser games.

If you do care about the technical details, read on!

Quite a while ago I [added closed caption support](http://toxicfrog.github.io/doomrl-ttysound/) to `doomrl-server`. This added visual indicators of sounds as they were played; given that sound is an important part of DoomRL, mechanically, the lack of sound when playing remotely was a serious hindrance. This brought remote play back into parity with local play, difficulty-wise; in fact, if anything it made it a bit easier, since you no longer had to worry about missing or mis-hearing sounds. However, the sound contributes a great deal of atmosphere to DoomRL, and that was still lacking.

About a year after writing that, I had an idea for how to add *actual* sound, and last weekend, I finally got around to implementing it.

A bit of a digression is called for here to discuss how the closed captions, and the terminal in general, work. All of the fancy colours and artwork drawn by DoomRL (and other terminal programs like `htop`, `nethack`, and even non-interactive programs like `ls` and `git` that output colour) is done entirely *in-band* -- all of the drawing information is contained entirely in the text sent to the terminal, rather than using a separate RPC channel or something of that ilk. This is done using *terminal control sequences*, sequences of characters that are not displayed to the user, but instead directly control the terminal -- repositioning the cursor, changing the foreground and background colour, etc. Since these are just ordinary ASCII text, they can be stored in files, transmitted over the network, and so forth, in the expectation that whatever terminal eventually displays them will understand them. This also means that you can do fancy terminal art with *anything* that knows how to output text; no special library support is needed!

Most terminals use a set of control sequences inherited from DEC's vt100 and vt220 serial terminals, ca. 1980; there's [a very comprehensive list for XTerm](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) and most other terminals adhere fairly closely to it, so it makes an excellent reference when working on this sort of thing. One thing you may notice is that a *lot* of these sequences start with the `ESC` character -- `\x1B` -- which is in fact the same character that's sent when you press the Escape key (assuming your window manager or whatever doesn't intercept it). And most of the sequences useful for artwork start with the two-character sequence `ESC [`, also known as *CSI*, Control Sequence Introducer. The most common of these is probably `CSI [ <a bunch of numbers> m`, which controls text style and foreground and background colour -- the numbers indicate what styles to use, while the trailing `m` indicates that this is a text styling command (as opposed to a command that moves the cursor, or clears part of the screen, or what have you). When the terminal sees one of these, it doesn't display it to the user, but instead changes how it will display text when next it *does* display something. For example, `ESC [ 1 m` enables bold.

(If you're interested in seeing this in practice, you can try something like `ls --color=always /dev | less -+r`; the `--color=always` is necessary because otherwise `ls` will detect it's not printing to the terminal and turn colour off, and the `-+r` tells `less` to display the "raw" control codes rather than passing them through to the terminal. Compare that to what happens when you pipe it through `less -R`!)

Anyways, this makes it pretty easy for the closed caption library to inject new graphics. The sound files are replaced with small text files that contain the symbols to display for each sound (including the control sequence characters for colour and style); and the sound library that DoomRL uses -- SDL_Mixer -- is replaced with a library that exports the same functions, but rather than playing the sounds through the speakers, just outputs a cursor positioning command to move the terminal to the bottom line, followed by the contents of the sound file. (This does leave the cursor at the bottom of the screen, but fortunately DoomRL is *very* aggressive about moving the cursor back into position before drawing, rather than assuming it stays where it was left.)

So, for example, the sound effect for "an imp taking damage" is rendered as a brown `i` bracketed with red `*`s: ![imp hit icon](/assets/doomrl-imp-hit.png). If we open up the `imp/hit` closed caption file, this is stored as (with spaces added for clarity):

    ESC [ 1;31 m * ESC [ 0;33 m i ESC [ 1;31 m *

Which is read by the terminal as: `CSI 1;31 m` (set style to bold (1) with red foreground (31)); `*` (display an asterisk); `CSI 0;33 m` (set style to default (0) with brown foreground (33)); `i` (display an i); and finally another `CSI 1;31 m *` to produce another bold red `*`. (Try this at home: `printf '\x1B[1;31m*m\n'` should display a bright red `*` in the terminal. If everything *stays* red, try `printf '\x1B[0m'` to reset things to normal.)

Now, as fun as this is -- and you can have a lot of fun with it! -- none of this gets us any closer to emitting actual sounds. Unfortunately, the closest thing we have to a standard way of emitting sounds in the terminal is the single-character command `BEL`, `\x07`. Back in the day this would have rung a physical bell attached to a teletypewriter; these days it tends to produce a desktop notification, screen flash, or beep. It's not much use for sound effects.

So, I introduced a new control sequence.

There's a bunch of other families of control sequences apart from the ones that start with CSI. In particular, `ESC ]` -- aka *OSC*, Operating System Commands -- is used to send commands that contain *text* rather than numbers. Unlike the CSI sequences, where what letter they end with determines what function they have, the OSC sequences *start* with a letter or number that determines the function, followed by `;` and some text, and finally `BEL` to mark the end of the command. For example, `ESC ] 2 ; ... BEL` sets the window title. What makes OSC in particular useful for this is:

- You can stick a lot of text into an OSC sequence, rather than just numbers as is the case with CSI;
- There's a lot of unused command space; xterm defines OSC commands from `OSC 0` up to `OSC 119`, which, even if we assume they're limited to three digits, still gives us nearly 800 unused command numbers to play with; and
- Terminals will reliably ignore OSC sequences that they don't understand.

That last one, in particular, means that if we co-opt OSC for sound control, terminals that don't understand our non-standard extensions will simply ignore them rather than displaying garbage or otherwise misbehaving.

So, I [modified the in-browser terminal](https://github.com/ToxicFrog/doomrl-server/blob/9f2163f126d0c45122ef2651c25c3dfacdec4b43/www/xterm2.js#L2483) that `doomrl-server` uses to recognize a new sequence: `OSC 666`. In particular, `OSC 666;1;<sound> BEL` will cause it to fetch `<sound>.flac` from the server and play it immediately, while `OSC 666;2;<track> BEL` will cause it to fetch `<track>` and play it using a separate music channel. No other terminal in the world understands these, but that's ok.

Once that was done, all that was missing was something on the server side to actually send the new sequences. A bit of work on the closed caption library to permit one file to contain multiple "captions" was necessary, but once done, it became possible to write closed caption files like this one (once again, the "imp takes damage" file):

    ESC [ 1;31 m * ESC [ 0;33 m i ESC [ 1;31 m *
    ESC ] 666;1;dspopain BEL

The first line is the same visual display from earlier, but the second line is the new `OSC 666;1` command. So now, when an imp takes damage, and the closed caption data for it is written to the terminal, that sequence will be sent along with the visual data -- most terminals will ignore it, but the in-browser terminal will understand it as an instruction to fetch `dspopain.flac` from the server and play it.

And since this is just text sent along with everything else, it also gets properly embedded in replays, allowing you to enjoy sound and music while spectating or reviewing recorded games, too.

Unfortunately, this will probably have to remain browser-only; I don't see the `xterm` or `konsole` team accepting patches to add support for such a niche use-case anytime soon. :)
