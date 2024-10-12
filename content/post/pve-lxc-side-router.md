---
title: PVE LXC OpenWRT  单网口旁路由设置
description: PVE LXC OpenWRT  单网口旁路由设置
date: 2024-02-22
image: https://upload.wikimedia.org/wikipedia/commons/8/84/OpenWrt_Logo.svg
categories:
    - HomeLab
tags:
  - proxmox
---

先下载openwrt，下载官方rootfs就行

```
https://downloads.openwrt.org/releases/23.05.2/targets/x86/64/

wget https://downloads.openwrt.org/releases/23.05.2/targets/x86/64/openwrt-23.05.2-x86-64-rootfs.tar.gz
```

上传到PVE, 创建模板：

```
pct create 101 local:vztmpl/openwrt-23.05.2-x86-64-rootfs.tar.gz --rootfs local-lvm:1 --ostype unmanaged --hostname OP --arch amd64 --cores 1 --memory 1024 --swap 0 -net0 bridge=vmbr0,name=eth0
```


编辑模板，加入最下面几行

```
features: nesting=1
lxc.include: /usr/share/lxc/config/openwrt.common.conf
lxc.cgroup2.devices.allow: c 108:0 rwm
lxc.mount.entry: /dev/ppp dev/ppp none bind,create=file
lxc.cap.drop: 
```

注意 `features: nesting=1`， 没有nesting会导致dnsmasq启动失败

最终看起来
```
arch: amd64
cores: 1
features: nesting=1
hostname: OP
memory: 1024
net0: name=eth0,bridge=vmbr0,hwaddr=BC:24:11:3B:3D:8D,type=veth
ostype: unmanaged
rootfs: local-lvm:vm-101-disk-0,size=1G
swap: 0
lxc.include: /usr/share/lxc/config/openwrt.common.conf
lxc.cgroup2.devices.allow: c 108:0 rwm
lxc.mount.entry: /dev/ppp dev/ppp none bind,create=file
lxc.cap.drop: 
```

之后启动，首先肯定连不上LuCI，因为网络设定以及防火墙设定，先修改  `/etc/config/network`

`vim /etc/config/network`

看起来像这样

```
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'fd3f:736d:11c4::/48'

config device
	option name 'eth0'

config interface 'lan1'
	option proto 'static'
	option device 'eth0'
	option ipaddr '192.168.101.100'
	option netmask '255.255.255.0'
	option gateway '192.168.101.1'
```

UI 设置，记得加入防火墙区域lan：

![[Pasted image 20240206155928.png]]

或者 `vim /etc/config/firewall`

```
config zone
	option name 'lan'
	option input 'ACCEPT'
	option output 'ACCEPT'
	option forward 'ACCEPT'
	option network 'lan1 utun'
```


然后重启，能访问UI了

### nftables 旁路由设置

大部分文档还停留在 `iptables`，现在OpenWRT都已经迁移到 `nftables` 了已经，设置应该如下

![[Pasted image 20240206161621.png]]


反正就是需要一个 `MASQUERADE`, 目标和源地址都任意。



### OpenWRT OPKG 设置

首先换成国内源：https://developer.aliyun.com/mirror/openwrt
命令：
```
sed -i 's_downloads.openwrt.org_mirrors.aliyun.com/openwrt_' /etc/opkg/distfeeds.conf
```

之后安装点必要的包

```
opkg update
opkg install luci-i18n-base-zh-cn luci-i18n-firewall-zh-cn luci-i18n-opkg-zh-cn
```


### OpenClash 安装

https://github.com/vernesong/OpenClash/wiki/%E5%AE%89%E8%A3%85
https://github.com/vernesong/OpenClash/releases

按以上两个链接安装OpenClash

由于 `dnsmasq` 会和 `dnsmasq-full` 撞车，于是先卸载dnsmasq. 之后再用下面的命令安装依赖

```
opkg install coreutils-nohup bash dnsmasq-full curl ca-certificates ipset ip-full libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
```


注意SCP现在走SFTP，需要加上 `-O`

```
scp -O ~/Downloads/clash_meta root@192.168.101.100:/etc/openclash/core
```

### OpenClash Side Note

踩了个坑，因为OpenClash的DNS不work。于是额外装了SmartDNS, 
设置DNS OverHTTPS, 使用 CloudFlare dns：
然后dnsmasq转发即可。

```
1.1.1.1/dns-query
```



## Ref

1. PVE LXC Openwrt 设置： https://dev.leiyanhui.com/openwrt/lxc-mian-op/
2. https://developer.aliyun.com/mirror/openwrt
3. https://github.com/vernesong/OpenClash/wiki/%E5%AE%89%E8%A3%85

