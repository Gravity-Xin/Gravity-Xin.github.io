---
title: Linux Shell ifconfig
date: 2018-12-27 20:31:12
categories:
- 编程技术
tags:
- ip
- mtu
comments: true
---

查看和修改网络接口配置信息
格式: ifconfig [网络设备] [命令参数与选项]

命令参数:

* up: 启动指定网络设备/网卡
* down: 关闭指定网络设备/网卡
* mtu [字节数]: 设置网络设备的最大传输单元
* netmask [子网掩码]: 设置网络设备的子网掩码
* broadcast [地址]: 设置网络设备的广播地址
* address [地址]: 设置网络设备的IPv4地址
* arp: 设置网络设备是否支持ARP协议

ifconfig修改的网络参数配置在系统重启之后失效，如果需要配置新永远生效，需要修改网卡的配置文件

实例:

```shell
ifconfig en0
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    ether f0:18:98:4e:79:0c
    inet6 fe80::53:26b5:4333:1d67%en0 prefixlen 64 secured scopeid 0xa
    inet 10.100.39.238 netmask 0xfffffc00 broadcast 10.100.39.255
    nd6 options=201<PERFORMNUD,DAD>
    media: autoselect
    status: active
```

en0: 网卡
UP: 网卡开启
RUNNING: 代表网卡的网线被接上
MULTICAST: 网卡支持组播
ether: 网卡的mac地址
inet6: ipv6地址
inet: ipv4地址
netmask: 子网掩码
broadcast: 广播地址