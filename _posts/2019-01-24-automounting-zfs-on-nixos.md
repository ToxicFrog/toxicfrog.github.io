---
layout: post
title: Automounting ZFS on NixOS
---

ZFS and NixOS are two great tastes that taste great together, _if_ you configure all of your ZFS datasets as `mountpoint=legacy` and list them in `configuration.nix`. Having done this, the NixOS boot scripts are able to import and mount anything mandatory for boot (notably, `/nix`), and systemd is made aware of the mountpoints and can ensure that before it starts a service, all the datasets that service uses are mounted.

If you want to use ZFS automounting, however, the wheels come off. That _does_ happen as part of the normal boot process, in `zfs-mount.service`. However, since systemd doesn't have any idea what mountpoints are owned by ZFS, it may try to start up some services before `zfs-mount` runs. If this happens, these services will, at best, fail, and at worst, will create blank configuration files or state directories or what have you, preventing `zfs-mount` from actually mounting everything.

You can try manually setting `Before/After` relationships between `zfs-mount.service` and the services that need ZFS mountpoints, but that's a lot of moles to whack. It's easy to get wrong, and if you do, it may not even fail consistency, since it's basically a race between service startup and zpool import. I tried this approach for months and never got it entirely working.

Fortunately, there's an easier, if somewhat brute-force, way.

First, get your mountpoints in order. Stuff needed by the stage1 boot process still needs to be in `configuration.nix`.

    $ zfs set mountpoint=legacy ancilla/root internal/nix
    $ nano /etc/nixos/hardware-configuration.nix

    {
      boot.zfs.devNodes = "/dev/disk/by-path";
      fileSystems."/" = { device = "ancilla/root"; fsType = "zfs"; };
      fileSystems."/nix" = { device = "internal/nix"; fsType = "zfs"; };
      ...
    }

This is sufficient to ensure that the initrd will include ZFS support, and will import the `ancilla` and `internal` pools and mount `/` and `/nix` before loading stage2. (Replace `/dev/disk/by-path` with `by-id` or similar if you prefer; I like `by-path` because it points me at the physical location of each drive, which is generally more useful to me than the serial number or whatnot.)

If, like me, you're using `by-path` and sometimes moving drives around, or (also like me) are using `by-id` with low-grade external docks that don't properly report the drive ID, you probably also want to disable the zpool import cache, which is only useful if the drives have stable names across boots:

    $ for pool in internal ancilla backup; do
        zpool set cachefile=none $pool
      done

Now we just insert a bit of code into the NixOS stage2 boot script. This will run just after the system profile you're booting has been activated, but just before systemd starts up, guaranteeing that your zfs datasets are mounted before any services start. (This is also a great place to insert things like calls to `sensors -s`, by the way.)

Note well that (a) this imports _all_ visible pools on boot, so if you attach a new pool before booting that will get imported _and mounted_, and (b) it won't hard-fail if it can't find a pool that used to be there. For me, both of these are desireable properties.

    boot.postBootCommands = ''
      ${pkgs.zfs}/bin/zpool import -a -N -d /dev/disk/by-path
      ${pkgs.zfs}/bin/zfs mount -a
    '';

Note that `$PATH` isn't set at this point, so we have to explicitly specify the path to the zfs package. (`echo` is fine because it's a shell builtin.)

"But wait!", you say. "Doesn't this run after activation, and thus, after home directories are created? Isn't that a problem if those home directories are on ZFS?" And you are quite right: it does, and it is.

In most cases, you can get around this by simply setting `createHome = false`. You can either create the home directory yourself, or leave it as true the first time you `nixos-rebuild` with that service enabled and then turn it off afterwards. For example, I have:

    users.users.airsonic.createHome = lib.mkForce false;
    users.users.deluge.createHome = lib.mkForce false;
    users.users.git.createHome = lib.mkForce false;

And similar settings for all the human users of the system. And in some cases, we don't even need to worry; IPFS, for example, initializes its home directory as part of service startup rather than in the activation script.

For me, the only recurring problem was `borgbackup`, which writes directory creation into the activation script by hand rather than using the `createHome` infrastructure (which makes sense, since the backup repo may be owned by a user with an existing home directory in a totally different location). But I also don't want to risk erasing my entire backup repo if, by some happenstance, it ends up mounted before the `postBootCommands` run. So, the final version:

    boot.postBootCommands = ''
      echo "=== STARTING ZPOOL IMPORT ==="
      # Clean up borg-repo because the activation script creates it
      ${pkgs.findutils}/bin/find /backup/borg-repo -maxdepth 1 -type d -empty -delete
      ${pkgs.zfs}/bin/zpool import -a -N -d /dev/disk/by-path
      ${pkgs.zfs}/bin/zpool status
      ${pkgs.zfs}/bin/zfs mount -a
      echo "=== ZPOOL IMPORT COMPLETE ==="
    '';

With this, any attached zpools that weren't imported in stage1 will get imported before systemd starts, and any ZFS filesystems with configured mountpoints will be automounted.
