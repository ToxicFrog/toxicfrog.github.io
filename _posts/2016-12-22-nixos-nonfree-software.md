---
layout: post
title: NixOS -- Software Installation
---

In general, there's four ways to install software in Nix:

* Using `nix-shell -p <package>`, which gives you a shell with an emphemeral install of the package;
* using `nix-env -i` as yourself;
* using `nix-env -i` as root;
* and editing `environment.systemPackages` in `configuration.nix`.

The first is useful for one-off tasks or experimenting with new packages, since the packages will be subject to garbage collection once you close the shell, but for this reason it's not good for things you use day to day.

The second installs the package just for you (and doesn't require root!). This lets ordinary users install packages just for themselves. The third is similar, but packages installed in this manner by root are available to all users. This is probably the closest to "traditional" package management, but like traditional package management the set of installed packages isn't stored in `~`, but in `/nix/var/nix/profiles/per-user/` -- so backing up someone's user directory won't back up their preferred set of installed packages! I find this alarming.

The last gets you a system-wide package install that is recorded in `/etc/nixos`, making it a convenient central location for version control and backups. For this reason I've been using `environment.systemPackages` exclusively for permanent package installs.

### Day to day programs

Most of the stuff I use on a daily basis is listed in the [package search](https://nixos.org/nixos/packages.html) and is straightforward to install. I needed `gitAndTools.gitFull` rather than just `git` to get stuff like `gitk`, but the `command-not-found` tool pointed me in the right direction. For the most part it's just a matter of adding the packages to `systemPackages` and then running `nixos-rebuild switch` to activate them.

### `hledger` and conflicting dependencies

I use `hledger` and `hledger-web` for (my attempts at) accounting. This is where some weirdness happened: `hledger-web` is version 1.0, and requires `hledger` 1.0, but `hledger` is 0.27.

####

By default, when you install NixOS, it's configured to install software from the `nixos-<release>` channel. This is the set of software at the time the release was cut + stability and security updates, but not generally any major version changes. A new release is cut every six months.

There's also the `unstable` channel, which tracks the current state of the `nixpkgs` repository -- modulo testing; commits that don't pass CI won't get pushed to the channel. This gives you a more rolling-release style experience, and like Tumbleweed, is sometimes exciting.



There's a whole pile of packages that I use on a daily basis, so all of those get added to `systemPackages`.
