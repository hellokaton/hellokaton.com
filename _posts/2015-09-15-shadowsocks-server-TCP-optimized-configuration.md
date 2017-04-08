---
layout: post
title: shadowsocks服务器TCP优化配置
categories: shadowsocks
tags: shadowsocks
---

该篇文章不支持openvz环境。

### 增加tcp连接数量

编辑limits.conf

```bash
vi /etc/security/limits.conf
```

增加以下两行

```bash
* soft nofile 51200
* hard nofile 51200
```

<!-- more -->

开启shadowsocks服务之前，先设置一下ulimit

```bash
ulimit -n 51200
```

### 调整内核参数

首先科普下TCP拥塞控制算法：
中美之间的线路质量不是很好，rtt较长且时常丢包。TCP的设计目的是解决不可靠线路上可靠传输的问题，即为了解决丢包，但丢包却使TCP传输速度大幅下降。HTTP协议在传输层使用的是TCP协议，所以网页下载的速度就取决于TCP单线程下载的速度（因为网页就是单线程下载的）。丢包使得TCP传输速度大幅下降的主要原因是丢包重传机制，控制这一机制的就是TCP拥塞控制算法。

Linux内核中提供了若干套TCP拥塞控制算法，这些算法各自适用于不同的环境。
1）reno是最基本的拥塞控制算法，也是TCP协议的实验原型。
2）bic适用于rtt较高但丢包极为罕见的情况，比如北美和欧洲之间的线路，这是2.6.8到2.6.18之间的Linux内核的默认算法。
3）cubic是修改版的bic，适用环境比bic广泛一点，它是2.6.19之后的linux内核的默认算法。
4）hybla适用于高延时、高丢包率的网络，比如卫星链路——同样适用于中美之间的链路。

我们需要做的工作就是将TCP拥塞控制算法改为hybla算法，并且优化TCP参数。

1、查看可用的算法。
主要看内核是否支持hybla，如果没有，只能用cubic了。

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

2、如果没有该算法，则加载hybla算法（不支持OpenVZ）

```bash
/sbin/modprobe tcp_hybla
```

3、首先做好备份工作，把sysctl.conf备份到root目录

```bash
cp /etc/sysctl.conf /root/
```

4、修改sysctl.conf配置文件，优化TCP参数

```bash
vi /etc/sysctl.conf
```

添加以下代码

```bash
fs.file-max = 51200
#提高整个系统的文件限制
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 3240000
 
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

5、保存生效

```bash
sysctl -p
```

### 小结

经测试，digitalocean,ramnode的KVM等内核支持hybla算法。但是linode的内核目前不支持，请参考Linode内核加载hybla模块进行加载。

需要注意的是每次重启需要重新加载hybla算法模块。

参考文章：[http://shadowsocks.org/en/config/advanced.html](http://shadowsocks.org/en/config/advanced.html)