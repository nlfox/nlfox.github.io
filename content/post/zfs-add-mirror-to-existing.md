---
title: ZFS Mirror vdev for existing storage
description: It's important to keep precious data properly backed up.
date: 2024-10-27
image: https://wiki.alopex.li/vdev.png
categories:
    - HomeLab
tags:
  - proxmox
  - zfs
  - raid
---


Recently I changed my NAS hardware, I found that as per my need, I don't actully need the orginal RAID-Z1 for the extra storage, instead I want my data to as secure as possible with mirror vdev (basically RAID1 in ZFS term). 

I re-ordered my storage architect to 3 layers. 

First layer is a SSD RAID1 zpool that runs Proxmox (HostOS) along with frequently accessed application data (like SQLite, VMDisk etc).

Second layer is HDD RAID1 zpool, holds all rest data that's not frequently accessed.

Third layer is a portable HDD serve as a cold backup for snapshots from both 1st and 2nd layer.

## Add vdev to mirror existing storage with proxmox boot

So if a disk is used for proxmox root mountpoint, it will be partitioned into 3: BIOS, EFI, and actual data.

So to add a SSD to existing proxmox pool, there are some extra work need to be done to copy the first 2 partition.  

Ref: https://pve.proxmox.com/wiki/ZFS_on_Linux

The first steps of copying the partition table, reissuing GUIDs and replacing the ZFS partition are the same. 


```bash
sgdisk <healthy bootable device> -R <new device>
sgdisk -G <new device>
```

Find the right disk:

```bash
ls -l /dev/disk/by-id | grep -v part
```

Then for the new partition 3, attach it to existing ZFS pool as mirroring vdev:

```bash
zpool attach rpool nvme-eui.<existing>-part3 nvme-eui.<new>-part3
```
Note that the order is important! Existing one should be the first argument, otherwise it might fail.

We only want to mirror partition 3 given it's the root filesystem.


Then use `proxmox-boot-tool` to initialize grub on it.

```bash
proxmox-boot-tool format /dev/nvme1n1p2
proxmox-boot-tool init /dev/nvme1n1p2 grub
```



















