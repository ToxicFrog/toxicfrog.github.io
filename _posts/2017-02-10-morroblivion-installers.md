SI:
http://www.nexusmods.com/oblivion/mods/10739/?

---
layout: post
title: Morroblivion Wizards
---

Morroblivion itself isn't too hard to install, but it can rapidly go out of control once you realize (a) how much better a few mods make the Oblivion UI and (b) how many bugfixes for Morroblivion exist that haven't been merged into the main ESP yet. (b) is especially pernicious because the Morroblivion forums have lots of threads about mods and fixes for it, but rarely if ever do they make it clear which ones are still useful and which ones have been obsoleted by subsequent Morroblivion releases.

Furthermore, even once you know what you want, you have to spend ages rummaging around on TESNexus to actually download the mods. And no-one should have to visit TESNexus.

So, I spent a while packing up the mods I play Morroblivion with and cleaning them up for Wrye Bash installation. All of these packs are Bash-compatible and most of them have wizards to let you tweak the install to your liking.

Here are my revised instructions for installing Morroblivion.

## Initial Setup

Before anything else, you'll need to download and install:

 * Morrowind, with the Tribunal and Bloodmoon expansions
 * Oblivion, with the Shivering Isles expansion
 * [Wrye Bash](https://github.com/wrye-bash/wrye-bash/releases) mod manager
 * [Oblivion Script Extender](http://obse.silverlock.org/)

OBSE can't be installed using Wrye Bash; just unpack it into your Oblivion directory. You should end up with `obse_loader.exe` and `Oblivion.exe` next to each other.

### Installing mods with Wrye Bash

Everything else linked from this page can be installed using Wrye Bash: start it up, click on the "installers" tab, then drag-and-drop the `.7z` file into the installers list to make Bash aware of it. To actually install it, right-click and choose `Wizard`, or, if that's not an option, `Install`.

## Prerequisites

Morroblivion requires some additional mods and OBSE plugins to function, and recommends installing the unofficial patches. These are packed together [here](http://funkyhorror.ancilla.ca/toxicfrog/tes/Morroblivion Prereqs.7z). Install it through Bash as noted above.

## Morroblivion proper

Having installed the prereqs, head over to the [Morroblivion release thread](http://tesrenewal.com/forums/morroblivion/mods/753) and download the master file installer and the resources. You need both!

Unpack and run the installer and it'll generate `morrowind_ob.esm` in your Oblivion directory. In addition to this file, you need

## enchanting

- bound boots/cuirass/gauntlets/greaves/helm/shield
- bound dagger/axe/bow/mace/spear/sword
- chameleon
- cure disease/paralysis/poison
- detect life
- detect magic
- fortify attack/attribute/fatigure/magic/health/skill
- fire/frost/lightning shield
- feather
- invisibility
- jump
- light
- levitate
- night-eye
- reflect damage
- restore fatigue/health
- reflect spell
- resist disease



## Morroblivion addons
## Oblivion addons
## Load order
## Included Mods

`Morroblivion Prereqs.7z` contains:

  * [Ely's Universal Silent Voice v0.93](http://www.nexusmods.com/oblivion/mods/16622/?)
  * [MenuQue v16b](http://www.nexusmods.com/oblivion/mods/32200/?)
  * [Unofficial Oblivion Patch v3.5.5](http://www.nexusmods.com/oblivion/mods/5296/?)
  * [Unofficial Shivering Isles Patch v1.5.9](http://www.nexusmods.com/oblivion/mods/10739/?)

`Morroblivion Addons.7z` contains:

  * Anhassi Fix
  * Morroblivion Fixes v1.5 (https://tesrenewal.com/forums/morroblivion-mod-releases/rel-morroblivion-fixes-v15)
  * Morroblivion Lost Magic v0.7
  * Morroblivion Magic Fixes v1.3
  * Morroblivion New Map v1.0 and v1.1
  * Morrowind Loading Screens
  * [Oblivion Magic Extender v1.0](http://www.nexusmods.com/oblivion/mods/31981/?)
  * Weather Fix

`Oblivion Addons.7z` contains:

  * [DarkUId DarN v1.6](http://www.nexusmods.com/oblivion/mods/11280/?)
  * [DarNified UI Configurator v1.4](http://www.nexusmods.com/oblivion/mods/34792/?)
  * Ely's Map Marker Be Done
  * Enhanced Hotkeys
  * Oblivion Stutter Remover
  * Pluggy v132-dev
  * Realistic Leveling
  * Toggleable Quantity Prompt


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
