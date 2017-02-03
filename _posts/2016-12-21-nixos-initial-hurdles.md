---
layout: post
title: NixOS: Installation & Initial Hurdles
---

The NixOS installer is, well, there isn't one as such; there's no guided installation program like you get with (say) SUSE or Ubuntu. The good news is that the actual process of installation is very quick and straightforward if you're comfortable managing partitions and mountpoints by hand:

* Boot your NixOS liveUSB
* Partition and format the target disk(s) and mount the target partitions on /mnt
* `nixos-generate-config --root /mnt`
* Edit /mnt/etc/nixos/*.nix to taste to customize the system you're about to install
* `nixos-install`
* Reboot

Despite the time spent looking up what various options in `/etc/nixos/configuration.nix` do, this was actually faster in the end than the SUSE installation wizard. I ended up doing a very minimal setup (mountpoints, hostname and hostid, root password) figuring that I could bootstrap the rest fairly easily.

### ZFS support

One very pleasant surprise this early on was [Nix's ZFS support](https://nixos.org/wiki/ZFS_on_NixOS). I was expecting at least some hackery would be needed to get ZFS working, but no -- add ZFS to `/etc/nixos/configuration.nix` in the liveUSB and `nixos-rebuild switch` to get the ZFS drivers in the live system, then create/import your zpools and mount `/mnt` as normal. The rest Just Works, and if you're using grub it'll automatically install a ZFS-compatible grub (so you can have both `/` and `/boot` on ZFS with no pain!) -- this is the configuration on my test server. Thoth uses EFI, so it needed a separate FAT32 `/boot` partition for the ESP, but everything else is ZFS.

### Networking, X11, User Account

So I boot into the installed system. It comes up with no problems whatsoever and extremely fast (no surprise, with no X and on z-mirrored SSDs). Then I realize I need a net connection to install more packages, but I have no idea how to operate the WLAN card from the command line, or if the tools to do so are even installed. So I scurry downstairs and briefly plug it into an actual network cable. Then it's time to set up a normal user account, a desktop environment, and wireless support:

    networking = {
      hostName = "thoth.ancilla.ca";
      hostId = "54485448";
      wireless.enable = true;
    };

    services.xserver = {
      enable = true;
      layout = "us";
      displayManager.sddm.enable = true;
      desktopManager.kde5.enable = true;
    };

    users.extraUsers.ben = {
      isNormalUser = true;
      shell = pkgs.zsh;
      extraGroups = ["wheel"];
    };

One `nixos-rebuild switch` later and I'm ready to go. (Almost. I'm using `mutableUsers` here, so I have to use `passwd` to set my password afterward. I could also have specified the password hash in `configuration.nix` if I weren't feeling lazy.)

### Wifi, Sound, Touchpad

This is where I start realizing that Nix defaults to *everything* off. If you want some behaviour, feature, or capability, you have to explicitly request it. In particular, the touchpad, sound, and wireless aren't working.

Wait, wireless? Didn't I set `networking.wireless.enable`?

This is where I realize that it's important to [read the documentation](https://nixos.org/nixos/options.html) rather than just looking at the name of each option. `networking.wireless.enable` enables `wpa_supplicant` and related tools, which I have no idea how to use. What I actually wanted was `networking.networkmanager.enable`. (This is one area where I'm not terribly pleased -- why is it `networking.wireless` rather than `networking.wpa_supplicant`? But it is nowhere near as bad as, say, Gentoo in this respect, and at least stuff is namespaced and well documented.) I also needed to add myself to the `networkmanager` group in order to have permission to configure the network.

Touchpad and sound support are easy to enable: `services.xserver.libinput.enable`, `sound.enable` and `hardware.pulseaudio.enable` are sufficient to get both working.

### zsh

I use `zsh` as my user shell, and set it appropriately above. But something is wrong -- the environment variables (including the custom `$PATH`) that make Nix function properly aren't being set up. Solution: I need to set `programs.zsh.enable` in addition to installing zsh. (I should probably send a patch to do this automatically if someone has zsh as their login shell). This is what enables the fancy `.zshrc` magic that makes everything work.

One note: `nix-shell` defaults to bash. `nix-shell --run zsh ...` will get you a zsh-based nix-shell while still (as far as I can tell) setting everything I need up correctly. The helpful people in `#nixos` inform me that this isn't the default (when your user shell is zsh) because there are some things it does that don't work right outside of bash, but they're related to nix package development, not general interactive use.

### chrome/chromium

I can't find Google Chrome in the [package list](https://nixos.org/nixos/packages.html) (for reasons that become apparent later), so I decide I might as well migrate to Chromium. This is just a matter of renaming `~/.config/google-chrome` to `~/.config/chromium` and adding `chromium` to `environment.systemPackages` in `configuration.nix`. (There are other ways to install packages in Nix, but I'm sticking with just putting everything in `systemPackages`.)

### \o/

And with that, I have a minimally useful system -- shell, wifi, web browser. The next step is to turn it into a *comfortable* one.
