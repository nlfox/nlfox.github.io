---
title: PVE 8.2 Setup Notes
description: Notes for issues encountered during my Proxmox VE 8.2 Setup.
date: 2024-10-24
categories:
    - HomeLab
tags:
  - proxmox
  - linux
  - grub
  - ventoy

---

## Download

I downloaded 8.2 from https://www.proxmox.com/en/downloads

## Bootable Device

I used Ventoy https://ventoy.net/en/index.html. The experience was great.

It's based on Grub2. Grub2 have support for ISO long ago, finally someone wrote some good wrapper.

Instead of dd, it can have multiple image and boot from any of it. But this actually caused me some issue, so for Proxmox try *NOT* to use Ventoy!


## Install ISO stuck `Waiting for /dev`

Using Ventoy, I boot into Proxmox VE ISO. But it stuck immediately 

```
"Waiting for /dev to be fully populated..."
```

It's due to the graphic driver. 

Press 'e' on the selection screen, GRUB will let you edit the start up parameters.

Removing after the `ro,quiet` parameter, add `nomodeset`. Press 'Ctrl + X' to boot selected entry. After that finally saw the install prompt.

## New install using ZFS as root won't start up

Another issue I encounter is after finished the install and rebooted, Proxmox stuck when loading the whole system. The message spamming whole screen didn't help, there is no error message at all. 


This could due to proxmox don't use classic GRUB2 as boot manager. The only way to force it install Grub2 is disable fast boot, enable legacy boot options and disable UEFI boot.

After re-install with above option, it should be able to boot properly.

## Remove subscription pop-up

Quick one-liner

```
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

## Remove paid Repo

Move `/etc/apt/sources.list.d/pve-enterprise.list` away.

```
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak
```

For `/etc/apt/sources.list.d/ceph.list`, my content:

```
deb http://download.proxmox.com/debian/ceph-quincy bookworm main
```

My `/etc/apt/sources.list`

```
deb http://ftp.us.debian.org/debian bookworm main contrib

deb http://ftp.us.debian.org/debian bookworm-updates main contrib

# security updates
deb http://security.debian.org bookworm-security main contrib
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

deb http://deb.debian.org/debian bookworm-backports main contrib non-free
deb-src http://deb.debian.org/debian bookworm-backports main contrib non-free
```