---
title: Delay mount until host is available
description: 
date: 2024-11-1
categories:
    - HomeLab
tags:
  - proxmox
  - linux
---

There is one problem where the VM that's sharing storage via NFS start up take time, another VM which takes dependency on the storage shared won't be able to mount properly using `/etc/fstab`. 

Unfortunately FSTAB don't have the flexibility to configure delay, so we need a script during start up.

### Mount script

```
#!/bin/bash

TARGET_IP="10.10.10.2"
MOUNT_POINT="/tank"
NFS_SHARE="10.10.10.2:/tank"

until ping -c 1 -W 1 "$TARGET_IP" > /dev/null 2>&1; do
    echo "Waiting for target IP $TARGET_IP..."
    sleep 5
done

mount -t nfs "$NFS_SHARE" "$MOUNT_POINT"
```

### Add to Systemd

Create a file in `/etc/systemd/system/mount-tank.service`

```
[Unit]
Description=Mount CIFS Share after Target IP is Available
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/root/mount-tank.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

### Delay docker start up 

Edit docker service: `systemctl edit docker.service`.

You can use a fix time delay like

```
[Service]
ExecStartPre=/bin/sleep 120
```

Or use a `RequiresMountsFor` option.

```
[Unit]
RequiresMountsFor=/tank/appdata/
```