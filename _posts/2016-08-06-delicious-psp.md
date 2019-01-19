---
layout: post
title: Making your PSP delicious
---

The PSP may be ten years old, but there's still a lot you can do with it. This post is documentation for my adventures with custom firmware (CFW) on the PSP.

## Why custom firmware?

### PSP games

Even if you're just playing PSP games and *nothing else*, it's worth installing CFW. The killer feature is the ability to dump UMDs to the memory stick and play them from there; not only do you not need to haul discs around everywhere, but you get faster load times, longer battery life, and no constant clicking and whirring from the optical drive. On top of that, plugins let you take screenshots, organize your games, apply unofficial patches, cheat, and even save and load state while in PSP games.

### PSX games

Yeah, you can play your PSX games on the PSP already -- if Sony has deigned to convert them, and you're willing to buy them again on PSN. With CFW, though, you can convert and play *any* of your PSX games, using freely available tools.

### Emulation

There are a lot of emulators available for the PSP. It'll replace your Gameboy {,Advance,Colour}, but why stop there? Make a dent in that backlog of SNES, NES, Sega Genesis, and Master System games! If you're adventurous you can even run some DOS and SCUMM games on it.

### Homebrew

The PSP homebrew scene tends to be focused more on utilities and emulators than on stand-alone games, but some of those utilities are pretty handy (like a built in file browser or the ability to push games to your PSP over the network), and it does have a small library of homebrew games as well, both original works and ports from other systems like Powder and Meritous.

## Basics

If you already know how to install games, ISOs, and PRX plugins, you can skip this section. Rather than reiterate install instructions for every piece of software below, I'm putting generic install instructions here.

### Games & Homebrew

PSP games consist of a directory containing the program file (`EBOOT.PBP`) and any other files needed. To install one, put the *directory* into `/PSP/GAME/`; you should end up with `/PSP/GAME/name_of_game/EBOOT.PBP`. (The actual name of the containing directory doesn't matter; the PSP will figure out the name of the program from the information in the EBOOT.) Once installed, the program will show up in the Games->Memory Stick XMB menu.

### ISOs

ISO and CSO (compressed ISO) images of UMD games, unlike EBOOTs, go in their own directory: `/ISO`. Once placed there, they'll show up alongside the EBOOTs.

### PRX plugins

Custom firmware has the ability to load plugins in PRX format. These add new features to the firmware. PRX files, along with any supporting data needed, go in the `/SEPLUGINS` directory. For the PSP to recognize them, though, you need to add a line to a configuration file; which file(s) you edit depends on what aspects of the system the plugin affects:

* `/SEPLUGINS/game.txt` for plugins that affect PSP games (both ISO and EBOOT)
* `/SEPLUGINS/pops.txt` for plugins that affect PSX games
* `/SEPLUGINS/vsh.txt` for plugins that affect the XMB

For each plugin, you'll need to add the line `ms0:/SEPLUGINS/path/to/plugin.prx 1`; replace the `1` with `0` if you want the plugin to be installed, but disabled.

Once installed, you can toggle plugins on and off from the recovery menu (`select` while in the XMB, then choose `Recovery Menu`).


## Setup

There are a bunch of different custom firmware options. These instructions are for installing the "PRO" CFW; it's the one I'm most familiar with, and one of the most widely used in general.

The latest official version of the PSP firmware is 6.61; however, due to the long gap (4 years!) between 6.60 and 6.61, and the fact that 6.61 adds no significant new features, most homebrew targets 6.60. So, although 6.61 PRO is available, this guide is for version 6.60 PRO-C2.

### Get the PSP ready

If you're already running firmware 6.60, you can skip this part. If you aren't, you need to be; the CFW installer patches your existing firmware, it doesn't install a complete firmware image from scratch.

If you're running an earlier version than 6.60, you can just download and run the [official 6.60 update](http://wololo.net/downloads/index.php/download/947). Install and run it like any other PSP game.

If you're on 6.61, you'll need a downgrader. Download FW 6.60 as above, and make sure you install it to `/PSP/GAME/UPDATE/EBOOT.PBP` so the downgrader can find it. Then download and install [Chronoswitch 6.1](http://wololo.net/talk/viewtopic.php?f=21&t=41227). Run it and it should automatically start the 6.60 installer to downgrade your PSP.

Once you're done here you can delete both the 6.60 update and Chronoswitch. They won't be needed again.

### Install custom firmware

Download the [6.60 PRO-C2](https://1a162edcad218a929d3e1c2d49e27469a8166389.googledrive.com/host/0BwYVUA7ansC5c09UbmZxTzVMNE0/6.60%20PRO-C2_16-12-2014.zip) installer. This comes with three PSP "games": `PROUPDATE`, `FastRecovery`, and `CIPL_Flasher`.

* `PROUPDATE` installs the custom firmware. Once installed, it is no longer needed. It's signed, so you can run it straight from the game menu even when running stock firmware (you didn't think that fancy DRM actually worked, did you?).
* `FastRecovery` is used after installation to load the CFW. You need to run it after every boot; if the PSP is fully shut down (not suspended), the CFW will be unloaded.
* `CIPL_Flasher` installs a CIPL (Custom Initial Program Loader) that automatically activates the CFW on boot. This only works on the PSP-1000, but lets you skip running `FastRecovery` on every boot. The drawback is that if, for some reason, you still want to run the stock firmware, you'll need to uninstall the CFW.

#### PSP-1000 with CIPL

Install `PROUPDATE` and `CIPL_Flasher`. Run both, in that order. Now you can delete them both; the PSP will boot straight into 6.60 PRO-C2 on startup.

#### PSP-2000/3000, or PSP-1000 without CIPL

Install `PROUPDATE` and `FastRecovery`. Run `PROUPDATE`. Once it's installed, you can delete it. After each complete shutdown, the PSP will start out running plain 6.60; run `FastRecovery` from the game menu to load the CFW. (Until you do so, the rest of your homebrew games, ISOs, and suchlike will either be missing from the XMB or show up as "corrupted data"; don't panic. They'll be back to normal once you run `FastRecovery`.)


## Install Plugins

### XMB Item Hider
### Game Categories Lite
### TempAR
### PSPStates

## Install Utilities

### PSP Filer
### PSP-FTPD

## Install Emulators

### NES: NesterJ
### SNES: snes9xTYLcm-mod
### GB/GBC & Master System: MasterBoy
### Genesis: PicoDrive
### GBA: UO gPSP Kai or TempGBA

gPSP works out of the box but has some issues (e.g. tutorials not displaying in Advance Wars 2). TempGBA works better but needs a BIOS. You can either use the open source BIOS at <fixme> or acquire an original GBA BIOS with md5sum a860e8c0b6d573d191e4ec7db1b1e4f6. Call it gba_bios.bin and put it in the same directory as the `EBOOT.PBP`.

## Install PSP games

### Dumping ISOs
### Where to put ISOs
### using cisoplus to generate CSOs from ISOs

## Install PSX games



### Using IceTea to convert PSX games
### multi-disc games
