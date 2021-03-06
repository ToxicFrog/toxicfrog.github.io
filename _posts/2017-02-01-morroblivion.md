---
layout: post
title: Setting Up Morroblivion
---

[Morroblivion](http://tesrenewal.com/) is a fan-made remake of Morrowind in the Oblivion engine. It works by running an installer that automatically converts the TES3 data to TES4 format, in conjunction with about 2GB of mod data that overhauls the textures, models, and so forth, either by replacing them with high quality originals or remapping them to Oblivion equivalents, and recreates feature from Morrowind that Oblivion doesn't support natively using the scripting system, like cast-on-use enchantments.

I'm writing this post mainly as a reminder to myself how to install it, since there are a few steps glossed over in the official instructions.

## Tools

I use [Wrye Bash](https://github.com/wrye-bash/wrye-bash/releases) for Oblivion mod management, and most of the mods I link to here either support WB or have been repacked to support it. To use a mod installer, go to the "installers" tab of WB, then drag-and-drop the mod archive onto it; once WB has loaded the installer, right-click and choose "Wizard" or, if that's not available, "Install".

Don't forget to go to the "mods" tab afterwards and enable the ESPs.

## Prerequisites

Morroblivion *requires*:

 * Morrowind, with the Tribunal and Bloodmoon expansions
 * Oblivion, with the Shivering Isles expansion
 * [Oblivion Script Extender](http://obse.silverlock.org/) -- unlike everything else here, OBSE can't be installed using Wrye Bash; just unzip it into your Oblivion directory so that `obse_loader.exe` and `oblivion.exe` are in the same directory.
 * MenuQue and Ely's Universal Silent Voice (both packed together [here](FIXME))

It is also *strongly recommended* that you install the [unofficial Oblivion patch](FIXME) and [unofficial Shivering Isles patch](FIXME).

## Morroblivion proper

Once you have those installed, head over to the [Morroblivion release thread](http://tesrenewal.com/forums/morroblivion/mods/753) and download the master file installer and the resources. You need both! The former will generate `morrowind_ob.esm`, the Morroblivion master file, from your Morrowind install; the latter can be installed via Wrye Bash and contains all the models, textures, and whatnot needed for it to function.

; notes
; first, install oblivion, morrowind, wrye bash, obse
; then "morroblivion prereqs" package, will automatically install OBSE modules and unofficial patches
; then put wizard.txt into morroblivion data package and install that; it'll ask for settings at install time
; then "morroblivion addons" for MO/magic/anhassi fixes, brighter nights, lost magic, loading screens, and map; OBME will be installed if needed
; WIP: then "oblivion addons" for TQP, enhanced hotkeys, realistic leveling, map marker be done, OSR, DarkUI, and DarNconfigurator


### Subpackages

When you select the Morroblivion installer in Wrye Bash, you'll see a list of "subpackages" that you can toggle on and off. Here's a quick rundown:

 * `00 Core`: the core mod; required.
 * `01 Better Map`: a much higher resolution version of the hand-drawn map that comes with Morroblivion. Note that this map both contains spoilers and is not entirely location-accurate; if either of those things bother you, don't install this (and look at "Optional Addons - Maps" below). If they *don't* bother you, though, this is definitely the prettiest map option.
 * `01 Chargen and Transport Mod` and `morrowind_ob: lets you choose whether to start in Seyda Neen or in the Imperial City, and travel between Morrowind and Oblivion. Choose this if you're importing an existing character, or want to play both Morrowind and Oblivion with the same character. If you only want to play Morrowind, don't bother.
 * `01 Morrowind Music`: installs the original Morrowind soundtrack.
 * `01 Tree Replacer`: replaces

## Prerequisites and load order

Morroblivion doesn't just require Shivering Isles at install time; you need to load `DLCShiveringIsles.esp` or a bunch of textures and meshes will be missing. Your load order should be something like:

 * `Oblivion.esm`, `Morrowind_ob.esm`, any other ESMs
 * `DLCShiveringIsles.esp`
 * Unofficial Oblivion and SI patches
 * `Morrowind_ob.esp` followed by the rest of the Morroblivion ESPs
 * Any other ESPs you're using

## Settings

`Data/ini/Morrowind_ob.ini` contains settings for Morroblivion proper. You can safely leave these at their defaults, but if you want the Full Morrowind Experience™, turn off fast travel and quest markers.

For in-game settings, I recommend turning *off* Distant Land and setting the draw distance to somewhere around 40-50%; this closely matches Morrowind's fogbound original feel. Setting the draw distance too far lets you see the wires and spoils at a lot of the atmosphere, IMO.

## Loading screens and intro

There's [a mod](https://tesrenewal.com/forums/oblivion/morroblivion-mod-releases/3776) replacing the Oblivion loading screens with Morrowind ones. There's also [a high res version](http://www.nexusmods.com/morrowind/mods/39329/?) of the intro FMV. You'll need to rename it `mw_intro.bik`.

## The map

The default map Morroblivion comes with is a low-res scan of the physical map that came with boxed copies of Morrowind. If you like this map, it also comes with an optional `Better Map` module; install and activate that and you get a much higher resolution version.

Personally, I *don't* like this map; the positions of cities on the map don't quite match up with their positions in game, and at the same time, it reveals the names and approximate locations of a bunch of spoiler-y things. I much preferred the very vague world map that came with the original. There doesn't seem to be an equivalent for Morroblivion, but the closest I've found is [New Morroblivion Map](http://www.nexusmods.com/oblivion/mods/40283/?) -- pick v1.1 if you want names on the map and v1.0 if you just want the unlabeled map. I prefer the latter, trusting the map markers that fill in as you explore to tell me what things are.

## Other Useful Mods

In general, mods that affect the UI or make changes to fundamental game mechanics (as opposed to mods that alter the worldspace or specific items, NPCs, or quests) will work fine in Morroblivion. So this means that [DarN UI](http://www.nexusmods.com/oblivion/mods/10763/?) and [DarK UI](http://www.nexusmods.com/oblivion/mods/11280/?) work fine (although you probably also want [the configurator](http://www.nexusmods.com/oblivion/mods/34792/?)), as does [Kyoma's Minimap](http://www.nexusmods.com/oblivion/mods/26220/?) (don't forget to set `bLocalMapShader=0` in `Oblivion.ini` if you want a full-colour map). [Enhanced Hotkeys](http://www.nexusmods.com/oblivion/mods/34735/?) is also a welcome addition (install it *after* DarN/DarK UI, if you're using one of them).

I've also installed [Map Markers Be Done](http://www.nexusmods.com/oblivion/mods/18501/?) (requires [Pluggy](http://www.nexusmods.com/oblivion/mods/23979/?)), which lets you tag map markers on the world map, and [Realistic Leveling](http://www.nexusmods.com/oblivion/mods/13879/?), a sort of spiritual successor to AF Leveling that, like AFL, dynamically calculates your stats and level based on your skills and eliminates a lot of the counterintuitive behaviours that the original Morrowind/Oblivion leveling system encourages, like deliberately dumping the skills you'll actually use at character creation to get more stat boosts per level up. (If installing RL with Wrye Base rather than OBMM, make sure to edit the INI; some of the settings don't have valid defaults because it expects OBMM to fill them in automatically at install time.)
