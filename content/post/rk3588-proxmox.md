---
title: RK3588 PVE 8.2 Setup Notes
description: I guess this is the last stop for HomeLab efficiency engineering...
date: 2024-10-29
categories:
    - HomeLab
tags:
  - proxmox
  - linux
  - arm
---

## Why Arm & Why RK3588

It's efficient with all the things I need, and it saves me a cup of coffee per month. Why not?

## Download system image

I use Armbian 
https://www.armbian.com/orange-pi-5-plus/

As recommanded, I use Etcher to flash image to TF card, you should be able to boot from TF card.

## Boot from M2 SSD

Use Etcher again to write to SSD, this way you will have a OS to boot from once remove TF card.

Execute `armbian-install`, and choose `Install/Update the bootloader on MTD Flash`

Once finished, you can power off and remove TF card.

## Install Proxmox VE base on armbian

Armbian is based on Debian Bookworm, so is a good base for Proxmox VE. Below guide contains everything needed for it to work.

https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm


## Start your first Arm64 VM on RK3588

Below is my config for my primary Ubuntu Server just for reference.
```
> # cat /etc/pve/nodes/orangepi5-plus/qemu-server/100.conf                                                     
affinity: 4-7
bios: ovmf
boot: order=scsi0;scsi2;net0
cores: 4
cpu: max
efidisk0: local:100/vm-100-disk-0.qcow2,efitype=4m,pre-enrolled-keys=1,size=64M
memory: 16384
net0: virtio=BC:24:11:20:D3:D2,bridge=vmbr0,firewall=1
net1: virtio=BC:24:11:95:6C:7A,bridge=vmbr1,firewall=1
numa: 0
onboot: 1
ostype: l26
scsi0: local:100/vm-100-disk-1.qcow2,iothread=1,size=32G
scsi1: local:100/vm-100-disk-2.qcow2,iothread=1,size=1T
scsi2: local:iso/ubuntu-24.04.1-live-server-arm64.iso,media=cdrom,size=2436748K
scsihw: virtio-scsi-single
sockets: 1
vga: ramfb
```

Arm KVM support is super limited, so the config is almost fixed.

CPU has to be `max`, BIOS has to be `OVMF`, vga has to be `ramfb`, SCSI and network driver has to be `VirtIO`.

CPU affinity must be set. KVM won't run if you mix A76 and A55 cores, so max CPU cores you can use for VM is 4.

Big core: 4-7, Small core: 0-3. In this VM I use all Big cores.

Side note: I do recommand general arm64 images instead of RK3588 specific for VM use, better compatibility, less compiling.


## Pass-through Devices

Short answer is, right now there is no great way to pass through devices in KVM given the limited IOMMU support. 

So the only way to pass-through GPU & NPU use mount bind in LXC container.

## Bind mount inside LXC container

```                                                                                                     
arch: arm64
features: nesting=1
hostname: CT101
memory: 16384
nameserver: 192.168.50.1
net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.50.1,hwaddr=BC:24:11:84:B6:82,ip=192.168.50.140/24,type=veth
net1: name=eth1,bridge=vmbr1,firewall=1,hwaddr=BC:24:11:5B:7E:AD,ip=10.10.10.3/24,type=veth
onboot: 0
ostype: ubuntu
rootfs: local:101/vm-101-disk-0.raw,size=32G
startup: order=2
swap: 4096
lxc.mount.entry: /dev/mali0 dev/mali0 none bind,optional,create=file
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/dma_heap dev/dma_heap none bind,optional,create=dir
lxc.mount.entry: /dev/rga dev/rga none bind,optional,create=file
lxc.mount.entry: /dev/mpp_service dev/mpp_service none bind,optional,create=file
```

Above is sample config that I used for bind mount to pass through a GPU & corresponding drivers.

## TBD: PCIE passthrough for RK3588

There is a actually a guide to enable PCIE passthrough, 

https://dl.radxa.com/users/dev/Rockchip_PCIe_Virtualization_Developer_Guide_CN.pdf

I don't need it that much, if I need it someday will try to make it work.
