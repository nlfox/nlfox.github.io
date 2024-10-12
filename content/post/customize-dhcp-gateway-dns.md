---
title: DHCP自定义设备网关和DNS
description: 毕竟很多时候让所有设备都走软路由不是个好主意。
date: 2024-10-11
categories:
    - HomeLab
    - Networking
tags:
  - proxmox
  - openwrt
  - dnsmasq
---

在设置完软路由（旁路由）之后，需要修改DHCP选项让软路由来做主要的路由。
但是大部分时候并不想让所有设备都走代理，一个一个设置静态ip太麻烦，而且有些设备并不能设置静态ip。

每个设备加入网络的时候会发一个DHCP广播(DISCOVER) 来请求ip。所以从原理上来讲，dhcp server肯定能通过不同的mac地址来分配DHCP地址。

以OpenWrt来举例，有几种办法来分配DHCP地址

### UCI 设置

```
uci set dhcp.mac1="mac"
uci set dhcp.mac1.mac="2A:AA:13:2C:D8:59"
uci set dhcp.mac1.networkid="vpn"
uci add_list dhcp.mac1.dhcp_option="3,192.168.50.10"
uci add_list dhcp.mac1.dhcp_option="6,192.168.50.10"
uci commit dhcp
```

请参考：https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#Options

`dhcp_option="3,192.168.50.10"`这个是网关
`dhcp_option="3,192.168.50.10"`这个是DNS

其实这些都被uci写进了 `/etc/config/dhcp`，之后需要修改就可以直接改文件。

### DNSMASQ 设置

其实没什么区别，反正提供DHCP的是DNSMASQ。


dnsmasq的配置文件是`/etc/dnsmasq.conf`。可以直接改。

```
dhcp-host=2A:AA:13:2C:D8:59,set:OTHER
dhcp-option=tag:OTHER,3,192.168.50.10
dhcp-option=tag:OTHER,6,192.168.50.10
```



