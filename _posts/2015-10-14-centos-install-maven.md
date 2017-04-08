---
layout: post
title: centos安装maven
categories: linux
tags: centos maven
---

Apache Maven，是一个软件（特别是Java软件）项目管理及自动构建工具，由Apache软件基金会所提供。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。曾是Jakarta项目的子项目，现为独立Apache项目。
　　那么，如何在Linux平台下面安装Maven呢？下面以CentOS平台为例，说明如何安装及配置Maven。

```sh
$ wget http://mirrors.sonic.net/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
$ tar xzf apache-maven-3.3.3-bin.tar.gz -C /usr/local/
$ cd /usr/local
$ ln -s apache-maven-3.3.3/ maven
```

<!-- more -->

上面的wget是从后面给定的URL中下载maven，当然，你也可以直接访问http://maven.apache.org/download.cgi手动下载。第二行命令是将下载下来的tar.gz包解压到/usr/local（tar默认将文件解压到当前目录，加了-C参数之后，是将解压的文件存放到/usr/local中）
　　当然，解压完下载下来的maven包是现在还不能启用，需要在PATH里面设置一下路径，如下：
　　
```sh
$ vim /etc/profile.d/maven.sh
```

```sh
export MAVEN_HOME=/usr/local/maven
export PATH=${MAVEN_HOME}/bin:${PATH}
```

设置好Maven的路径之后，需要运行下面的命令

```sh
source /etc/profile.d/maven.sh
```

使得上面设置的环境变量立即生效。
你也就可以重启一下电脑，使得上面的环境变量立即生效，但是没有上面的命令来得快！
当然，你也可以在/etc/profile文件后面加入下面三行，和上面的一样效果

```sh
$ vim /etc/profile
```

```sh
MAVEN_HOME=/usr/local/maven
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

设置好Maven的路径之后，需要运行下面的命令

```bash
source /etc/profile
```

使得上面设置的环境变量立即生效。
弄完之后，你可以运行下面的命令。

```bash
mvn -v
```

```bash
Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T19:57:37+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_60, vendor: Oracle Corporation
Java home: /usr/local/java/jdk1.8.0_60/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-431.23.3.el6.x86_64", arch: "amd64", family: "unix"
```

如果出现了上面类似的字段，说明Maven安装及配置完了！
上面的命令为了方面，都是在root用户下进行操作的，这样很不安全，建议使用一般的用户权限配合sudo去安装和配置！
