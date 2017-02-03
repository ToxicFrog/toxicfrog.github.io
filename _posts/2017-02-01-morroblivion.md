---
layout: post
title: Setting Up Morroblivion
---

[Morroblivion](http://tesrenewal.com/) is a fan-made remake of Morrowind in the Oblivion engine. It works by running an installer that automatically converts the TES3 data to TES4 format, in conjunction with about 2GB of mod data that overhauls the textures, models, and so forth, either by replacing them with high quality originals or remapping them to Oblivion equivalents, and recreates feature from Morrowind that Oblivion doesn't support natively using the scripting system, like cast-on-use enchantments.

I'm writing this post mainly as a reminder to myself how to install it, since there are a few steps glossed over in the official instructions. (I'm using Wrye Bash for this.)

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
