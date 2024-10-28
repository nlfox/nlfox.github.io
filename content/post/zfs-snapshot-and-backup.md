---
title: Note for ZFS Snapshot and Backup
description: It's important to keep precious data properly backed up.
date: 2024-10-26
image: https://lowtek.ca/roo/wp-content/uploads/2024/04/openzfslogo.png
categories:
    - HomeLab
tags:
  - proxmox
  - linux
---


### Create ZFS on backup external disk

Find the drive from `/dev/disk/by-id/`

```
> ls /dev/disk/by-id/
...
usb-Seagate_Expansion_HDD_00000000XXXXX-0:0
...
```

Create Backup zpool

```
zpool create seagate_backup_pool /dev/disk/by-id/usb-Seagate_Expansion_HDD_00000000XXXXX-0:0
```

### Import Zpool in external drive

Command `zpool import` without any argument will show what's available for import in current system.

After that, do `zpool import seagate_backup_pool -f` to actually import the backup pool.


### Take Snapshot of current system

Check current snapshot
```
zfs list -t snapshot
```

Take snapshot for all dataset.
```
zfs snapshot -r tank@current
```

### Send snapshot to backup pool

Backup as Dataset

```
zfs send -R tank@current | zfs recv seagate_backup_pool/pve-init-10-24-2024
```

Increment
```
zfs send -i -R tank@current | zfs recv seagate_backup_pool/pve-init-10-24-2024
```

Backup as single file

```
zfs send -R tank@current > tank.bak
```

### Make USB external drive Sleep

Install hd-idle and execute:

To instantly test out:

```
hd-idle -t sdd
```

hdparm won't work for usb external drive. 

To check drive status:
```
hdparm -C /dev/sd[abc]
smartctl -i -n standby /dev/sda
```

To reguarly run `hd-idle`, use it's own service.

`vim /etc/default/hd-idle`

Below opts will make `hd-idle` sleep device after 600s if no activity.

```
HD_IDLE_OPTS="-i 600 -l /var/log/hd-idle.log"
```

