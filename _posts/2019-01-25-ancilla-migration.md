---
layout: post
title: Ancilla hardware migration
---

I recently migrated the family server, `ancilla`, to new hardware and new disks. This post is primarily a record of that process. Hopefully it will be helpful to someone; I suspect it will, at least, be useful to Future Me when planning the next migration.

It has also been edited somewhat to remove false starts and dead ends, along with several hours of swearing and swapping drives around necessitated by one of my USB drive enclosures failing halfway through a resilver.

To reduce ambiguity, I'm going to name every machine and disk discussed here. The old server is `ancilla-old`, and the new one is `ancilla-new`; disks in the old one all have names starting with `dO`, and in the new one, with `dN`.

To set the stage, the original server (`ancilla-old`) has an external four-disk raidz5 holding most of the system; one internal SSD with `/boot` and `/nix` on it, as well as serving as the ZIL (journal) and L2ARC (cache) for the raidz5 and as swap; and an external two-disk mirror used for storing local backups and as scratch space. In detail:

    zpool ancilla
      raidz5 dObig1 dObig2 dObig3 dObig4
      filesystems / /home /etc ...
      zil dOssd-part2
      l2arc dOssd-part5
    zpool internal
      disk dOssd-part4
      filesystems /boot /nix
    zpool backup
      mirror dOsmall1 dOsmall2
    swap dOssd-part3

`ancilla-new` doesn't have any data on it yet, just a bunch of internal disks:

    m.2: dNssd
    sata: dNbig1 dNbig2 dNbig3 dNbig4

The goal here is to move everything on `ancilla` and `internal` onto the internal disks in the new ancilla, and move `backup` onto the `dObig*` disks, with a minimum of downtime -- no sitting there for ages booted off a liveUSB while `dd` runs, for example. At the same time, I want to switch from MBR booting to UEFI.

So. Step one, shut down `ancilla-old` and physically move all the disks over. The external enclosures are disconnected from the old and connected to the new, and the internal SSD is removed and attached to the new system via a single-disk dock. So `ancilla-new` now looks like this:

    m.2: dNssd
    sata: dNbig1 dNbig2 dNbig3 dNbig4
    usb: dObig1 dObig2 dObig3 dObig4
    usb: dOsmall1 dOsmall2
    usb: dOssd

Now we fire it up and boot (in MBR mode) from `dOssd`...and up it comes! The `postBootCommands` discussed in the [previous post](http://toxicfrog.github.io/automounting-zfs-on-nixos/) ensure that all the zpools are correctly imported even though the devices have moved around.

My first priority now is to remove the dependency on the old SSD and switch to UEFI booting.

    $ parted dNssd
        [create some partitions:]
        part1 /boot, 1GB
        part2 ZIL, 1GB
        part3 swap, 16GB
        part4 /nix, 64GB
        part5 L2ARC, everything else
    $ zpool add ancilla log dNssd-part2
    $ zpool add ancilla cache dNssd-part5

The old ZIL and L2ARC can be removed immediately:

    $ zpool remove ancilla dOssd-part2
    $ zpool remove ancilla dOssd-part5

Similarly, swap doesn't require any transition period, we just turn on the new and turn off the old:

    $ mkswap -L m2-swap dNssd-part3
    $ swapon dNssd-part3
    $ swapoff dOssd-part3
    $ nano /etc/nixos/hardware-configuration.nix

    {
      swapDevices = [ { device = "/dev/disk/by-label/m2-swap"; } ];
    }

And the zpool itself is almost as easy to deal with. We just add the new disk as a mirror of the old, wait for the resilver, and then remove the old disk. The final `zpool online -e` command tells it to expand the pool to fill the entire device.

    $ zpool attach internal dOssd-part4 dNssd-part4
    ...wait a bunch...
    $ zpool detach internal dOssd-part4
    $ zpool online -e internal dNssd-part4

The only part that requires more care here is `/boot`, since we're switching from MBR boot with `/boot` on ZFS to UEFI boot with `/boot` (of necessity) on FAT32.

    $ mkfs.fat -F 32 -n BOOT dNssd-part1
    $ nano /etc/nixos/...

    {
      fileSystems."/boot" = {
        device = "/dev/disk/by-label/BOOT"; fsType = "vfat";
      };
      boot.loader.grub = {
        enable = true;
        efiSupport = true;
        efiInstallAsRemovable = true;
        version = 2;
        device = "nodev";
      };
    }

Of particular note here are `efiInstallAsRemovable = true`, which tells GRUB to install itself at `esp:/EFI/BOOT/BOOTX64.EFI`, guaranteeing that the system will find it at boot even if there are issues updating the EFI boot variables, and `device = "nodev"`, which tells GRUB not to even attempt installing an MBR-compatible stage1 bootloader.

Now we just mount the new `/boot` and update it:

    $ zfs set mountpoint=none internal/boot
    $ mount /dev/disk/by-label/BOOT /boot
    $ nixos-rebuild boot

If all has gone well, `/boot/EFI`, `/boot/grub`, and `/boot/kernels` should have been created and now contain the new (UEFI-compatible) boot configuration.

And with that the last dependency on the old SSD is gone, so we can go ahead and unplug it.

The rest is very straightforward. First of all, we move the `ancilla` zpool to the internal disks. At first I thought I could create mirrors from each one, then detach the old disks, similar to how `internal` was handled; unfortunately, ZFS requires that raidz vdev be composed of individual block devices -- you can't have a raidz-of-mirrors. Fortunately, it has the `replace` command for this use case:

    $ zpool replace ancilla dObig1 dNbig1
    $ zpool status ancilla
      ...
      raidz1-0
        replacing-0
          dObig1     ONLINE
          dNbig1     ONLINE (resilvering)
        dObig2       ONLINE
        dObig3       ONLINE
        dObig4       ONLINE

Once `dNbig1` is fully resilvered into the array, it automatically detaches `dObig1` and we replace the process with the other three drives. (We could do all four drives at once, but that will be slower overall than doing them one at a time; as more of the drives are moved onto the faster individual SATA connections, the performance of each subsequent resilver increases dramatically.)

Now the `dObig*` disks are unused, and we can move the `backup` pool onto them using the same mechanism as `internal` (since it's already a mirror):

    $ zpool attach backup dOsmall1 dObig1
    ...wait for resilver...
    $ zpool attach backup dOsmall1 dObig2
    ...wait for resilver...
    $ zpool detach backup dOsmall1
    $ zpool detach backup dOsmall2
    $ zpool online -e backup dObig1

And there we go! One reboot, a few minutes of downtime, a few days of resilvering, and all the data is warm and snug in its new home. And I have four extra drives (`dObig3-4` and `dOsmall1-2`) I need to figure out what to do with.
