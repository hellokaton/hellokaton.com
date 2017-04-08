---
layout: post
title: 升级python2.6到2.7并安装pip
categories: python
tags: python pip
---

CentOS系统默认的python版本是2.6，目前很多python的操作在2.7版本会有好的支持，那么这里手动进行升级。

## 1. 升级Python

使用 `wget` 下载python2.7

```sh
[root@localhost ~]# wget https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz
[root@localhost ~]# tar -zxvf Python-2.7.10.tgz
[root@localhost ~]# cd Python-2.7.10
[root@localhost Python-2.7.10]# ./configure --enable-shared --with-zlib
```

之后执行

```sh
[root@localhost Python-2.7.10]# vi ./Modules/Setup
```

<!-- more -->

找到 `#zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz` 去掉注释保存，然后编译安装

```sh
[root@localhost Python-2.7.10]# make && make install
```

等待安装完成...

安装好 `Python2.7` 之后我们需要先把  `Python2.6` 备份起来，然后再对yum的配置进行修改，如果不进行这一步操作的话，执行yum命令将会提示你Python的版本不对。

执行以下命令，对 `Python2.6` 进行备份，然后为 `Python2.7` 创建软链接

```sh
[root@localhost ~]# mv /usr/bin/python /usr/bin/python2.6.6
[root@localhost ~]# ln -s /usr/local/bin/python2.7 /usr/bin/python
```

然后编辑 `/usr/bin/yum` ，将第一行的 `#!/usr/bin/python` 修改成 `#!/usr/bin/python2.6.6`
现在执行yum命令已经不会出现之前的错误信息了。

我们执行 `python -V` 查看版本信息，如果出现错误

```sh
[root@localhost ~]# python -V
python: error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory
```

编辑配置文件 `vi /etc/ld.so.conf`

添加新的一行内容 `/usr/local/lib`，保存退出，然后

```sh
/sbin/ldconfig
/sbin/ldconfig -v
```

再次执行 `python -V` 显示为

```sh
[root@localhost ~]# python -V
Python 2.7.10
```

## 2. 安装pip

下载最新版的pip，然后安装

```sh
[root@localhost ~]# wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

查找pip的位置

```sh
whereis pip
```

找到 `pip2.7` 的路径，为其创建软链作为系统默认的启动版本

```sh
ln -s /usr/local/bin/pip2.7 /usr/bin/pip
```

pip安装完毕，现在可以用它下载安装各种包了 :)

