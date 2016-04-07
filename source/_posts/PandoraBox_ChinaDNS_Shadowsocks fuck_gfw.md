# PandoraBox 配合ChinaDNS,Shadowsocks实现智能翻墙
title: PandoraBox 配合ChinaDNS,Shadowsocks实现智能翻墙
date: 2016-04-07 21:00
categories:
- notes
tags:
- Fuck GFW
- shadowsocks
- openwrt
- pandorabox
------

>必备:
>openwrt或者PandoraBox路由器一个（推荐PandoraBox，内置Shadwosocks,ChinaDNS）,
>Shadowsocks账号
>刷路由器在这里就不写了，google一大堆教程

## 1. 设置Shadowsocks

PandoraBox系统内置Shadwosocks,ChinaDNS等服务，十分方便，如果是openwrt系统要自己安装

### 1. 解决dns污染的问题
打开 `服务`>`ChinaDNS`

<!--more-->

![ChinaDNS设置](http://7xip25.com1.z0.glb.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20160407202953.png)

>之前就是这里设置错了没在意看，到时只能上去Google，却上不去Youtube,Facebook等（Google还没被DNS污染之前，现在已经加入DNS污染全家桶了）。
### 2. 设置Shadowsocks

打开 `服务`>`Shadowsocks`

![Shadowsocks设置](http://7xip25.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160407203552.png)

>填好配置文件，勾选透明代理一项，不要勾选Socks5代理。如果服务器支持UDP转发，可以勾选UDP转发一项，保存

### 3. 设置DNS转发

打开 `网络`>`DHCP/DNS`

![DNS设置](http://7xip25.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160407204044.png)

把DNS转发一项设置为127.0.0.1#1053(1053是第一步里面ChinaDNS设置的端口)

打开 `网络`>`DHCP/DNS`>`HOSTS和解析文件`选项卡,`勾选忽略解析文件一项 `

保存即可。