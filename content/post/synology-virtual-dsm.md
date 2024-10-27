---
title: Synology Virtual DSM
description: Notes for setting up Virtual DSM to backup 
date: 2024-10-25
categories:
    - HomeLab
tags:
  - proxmox
  - linux
  - synology
---

## Enable KVM on virtual machine

Switch virtual server CPU type to match host. Somehow this option not exist on UI
```
qm set 105 --cpu host
```




## Config

```yaml
services:
  dsm:
    container_name: dsm
    image: vdsm/virtual-dsm
    environment:
      DISK_FMT: "qcow2" # Root disk will be a single file in qcow2, recommanded
      RAM_SIZE: "2G"
      CPU_CORES: "2"
    devices:
      - /dev/kvm
    cap_add:
      - NET_ADMIN     
    volumes:
      - /tank/appdata/vdsm/data:/storage # need a local storage for its data
    stop_grace_period: 2m
    ports:
      - 127.0.0.1:5022:22 # Local port open for SSH access
      - 5000:5000
```


## Make Synology able to index and access remote FS.

```
sudo -i
wget http://code.imnks.com/face/PatchELFSharp
chmod +x PatchELFSharp
./PatchELFSharp "/usr/lib/libsynosdk.so.7" "SYNOFSIsRemoteFS" "B8 00 00 00 00 C3"
```

We use host network so need to specify `ssh 127.0.0.1 -p 5022`.


### Ref

https://wilsonzeng.com/article/c50e99ee-ab33-4da0-9c64-6cb4d0462f94