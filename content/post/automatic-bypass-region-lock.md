---
title: "自动地域分流代理"
description: "如果只靠IPGeo规则判断明显是不够的。"
date: 2024-10-20
image: 
categories:
    - HomeLab
    - Networking
tags:
    - proxmox
    - openwrt
    - dnsmasq
    - clash
---


我对翻回墙内的需求很少，但是最近有些奇怪的app有地区锁，于是就顺便把在国内翻墙的东西给反向拿出来了，记录一下，for my own record。

## How and Why

首先，基本的想法是，我们需要两级的分流，第一级是DNS。现代的网站通过DNS来进行分流以及负载均衡的情况非常普遍。如果只靠IP地理位置来走代理，很容易出现的情况就是，由于DNS查询的地区和实际访问的地区不同，你并不会被DNS分配到想要的endpoint。例如，某个qq.com的子域名，如果用美国IP访问，会返回香港的CDN地址，绕过了IP地址代理规则，导致绕过地域锁失败。


## Where to find rule data

当然我们指望自己写规则不现实，所以需要一些第三方维护的规则文件。我选用的是下面这个源：

https://github.com/Loyalsoldier/v2ray-rules-dat



## Solutions

现在有很多种成熟的一体化解决方案，我自己用的是Clash。


### Clash 配置

Clash会接管DNS，所以可以做到DNS分流。并且由于Clash也自带了以上的数据库，所以配置起来比较简单。

```yaml
dns:
  enable: true
  listen: 0.0.0.0:53
  nameserver:
    - tls://8.8.4.4 # GoogleDNS
    - tls://208.67.222.222 # OpenDNS
  nameserver-policy:
    geosite:cn:
      - 114.114.114.114
rules:
  - GEOSITE,cn,Proxy
  - GEOIP,cn,Proxy
```

GEOSITE负责的是中国大陆的域名。 DNS的部分的大意就是默认走GoogleDNS or OpenDNS，如果DNS请求符合GEOSITE:CN，就使用114的DNS来解析。而114DNS由于是中国大陆的IP，会走中国大陆的代理来访问，保证获得正确的结果。

当然凡事都会有例外，比如我遇到的是某个屎黄色论坛把我代理IP给ban了，于是我就在rules顶端加了一行规则让屎黄色论坛走直连。

```yaml
  - DOMAIN-SUFFIX,178.com,DIRECT
```



