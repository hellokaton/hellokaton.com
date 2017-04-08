---
layout: post
title: linux ssh连接不了问题
categories: linux
tags: centos
---

最近在VMWare上装了一台centos6的机器，装好基础环境后重启，ssh连接的时候发现连接不上去

![](https://i.imgur.com/y8xFRm0.png)

在xshell中提示如上图，于是在虚拟机中登录看看。

<!-- more -->

首先试试：

```sh
ssh localhost
```

提示

```sh
Read from socket failed: Connection reset by peer
```

于是在网上找了一大圈，木有解决，大部分结论是重新生成rsa和dsa

操作如下：

```sh
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key 
ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
```

http://askubuntu.com/questions/205179/ssh-problem-read-from-socket-failed-connection-reset-by-peer

我试了后，然并卵，在重启 sshd 的时候发现提示了一句话：

```sh
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Permissions 0755 for '/etc/ssh_host_key' are too open.

It is recommended that your private key files are NOT accessible by others.

This private key will be ignored.

bad permissions: ignore key: /etc/ssh_host_key

Could not load host key: /etc/ssh_host_key

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Permissions 0755 for '/etc/ssh_host_rsa_key' are too open.

It is recommended that your private key files are NOT accessible by others.

This private key will be ignored.

bad permissions: ignore key: /etc/ssh_host_rsa_key

Could not load host key: /etc/ssh_host_rsa_key

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Permissions 0755 for '/etc/ssh_host_dsa_key' are too open.

It is recommended that your private key files are NOT accessible by others.

This private key will be ignored.

bad permissions: ignore key: /etc/ssh_host_dsa_key

Could not load host key: /etc/ssh_host_dsa_key

Disabling protocol version 1. Could not load host key

Disabling protocol version 2. Could not load host key

sshd: no hostkeys available -- exiting.
```

意思就是说ssh 权限设置错误导致的问题，那么解决方法就好找了。

```sh
cd /etc/ssh
chmod 0644 *
chmod 0600 ssh_host_dsa_key ssh_host_key ssh_host_rsa_key
service sshd restart
```
