---
layout: post
title: centos安装tomcat
categories: linux
tags: centos tomcat
---

这篇文章将介绍安装和基本配置Tomcat 8的CentOS 5.x或CentOS 6.x

Tomcat8实现jsp2.2和Servlet 3.0规范和大量的新功能。访问管理器应用程序比起6x也有一个新的外观和细粒度的角色

在这篇文章中,我们将安装Tomcat8,新JDK7配置Tomcat作为服务,创建一个启动/停止脚本,以及(可选)配置Tomcat运行在非ROOT用户。

我们还将配置基本访问Tomcat Manager和快速使用`JAVA_OPTS`看看内存管理

最后,我们将看看在80端口上运行Tomcat以及一些策略。

首先,我们需要安装Java开发工具包(JDK)8
Tomcat8要求JDK版本最低为1.6。

<!-- more -->

### 第一步:安装JDK 1.8

你可以在这里下载最新的JDK: [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

我们将安装较新的jdk,`jdk-8u25`

我的操作系统是CentOS6.5_x64，这里选择是的:jdk-8u25-linux-x64.tar.gz
如果你是32位系统，请选择jdk-8u25-linux-i586.tar.gz

首先创建一个目录 `/usr/java`:

```bash
[root@srv6 ~]# mkdir /usr/java  
```

进入到`/usr/java`

```bash
[root@srv6 ~]# cd /usr/java    
[root@srv6 java ]#   
```

下载合适的JDK并将其保存到`/usr/java`目录。

解压`jdk-8u25-linux-x64.tar.gz`到`/usr/java`目录，使用命令tar -xzf:

```bash
[root@srv6 java]# tar -xzf jdk-8u25-linux-x64.tar.gz
```

这里会创建`/usr/java/jdk1.8.0_25`，这个是 `JAVA_HOME`

我们现在可以设置JAVA_HOME并将它加入环境变量

```bash
[root@srv6 java]# JAVA_HOME=/usr/java/jdk1.8.0_25
[root@srv6 java]# export JAVA_HOME  
[root@srv6 java]# PATH=$JAVA_HOME/bin:$PATH  
[root@srv6 java]# export PATH  
```

将JAVA_HOME设置为永久，我们需要在`~/.bash_profile`添加,也可以配置`/etc/profile`给所有用户

```bash
JAVA_HOME=/usr/java/jdk1.8.0_25
export JAVA_HOME  
PATH=$JAVA_HOME/bin:$PATH  
export PATH  
```

设置了`~/.bash_profile`后退出重新登录测试是否正确的设置了JAVA_HOME

```bash
[root@srv6 ~]#  echo $JAVA_HOME  
/usr/java/jdk1.7.0_05  
```

### 第二部：下载并解压Tomcat8

将tomcat8安装在`/usr/share`下

切换到`/usr/share`目录:

```bash
[root@srv6 ~]# cd /usr/share  
[root@srv6 share ]#   
```

下载tomcat8:http://mirror.tcpdiag.net/apache/tomcat/tomcat-8/v8.0.23/bin/apache-tomcat-8.0.23.tar.gz
并解压到`/usr/share`

使用`tar -xzf`解压：

```bash
[root@srv6 share ]# tar -xzf apache-tomcat-8.0.23.tar.gz
```

这将创建一个目录`/usr/share/apache-tomcat-8.0.23.tar.gz`

### 第三步:配置Tomcat作为服务运行

现在,我们将看到如何运行Tomcat作为服务和创建一个简单的启动/停止/启动脚本,以及在引导启动Tomcat。

切换到`/etc/init.d`目录创建一个tomcat的脚本：

```bash
[root@srv6 share]# cd /etc/init.d  
[root@srv6 init.d]# vi tomcat  
```

下面是我们使用的脚本：

```bash
#!/bin/bash  
# description: Tomcat Start Stop Restart  
# processname: tomcat  
# chkconfig: 234 20 80  
JAVA_HOME=/usr/java/jdk1.8.0_25 
export JAVA_HOME  
PATH=$JAVA_HOME/bin:$PATH  
export PATH  
CATALINA_HOME=/usr/share/apache-tomcat-8.0.23
  
case $1 in  
start)  
sh $CATALINA_HOME/bin/startup.sh  
;;   
stop)     
sh $CATALINA_HOME/bin/shutdown.sh  
;;   
restart)  
sh $CATALINA_HOME/bin/shutdown.sh  
sh $CATALINA_HOME/bin/startup.sh  
;;   
esac      
exit 0  
```

上面的脚本非常简单，包含你需要的基本元素

正如你看到的，我们只需要调用`startup.sh`和`shutdown.sh`，sh脚本位于tomcat的bin目录

你可以根据需要调整脚本

CATALINA_HOME是Tomcat的家目录(/usr/share/apache-tomcat-8.0.23)

现在给脚本授权

```bash
[root@srv6 init.d]# chmod 755 tomcat  
```

我们使用chkconfig启动tomcat

```bash
[root@srv6 init.d]# chkconfig --add tomcat  
[root@srv6 init.d]# chkconfig --level 234 tomcat on  
```

验证：

```bash
[root@srv6 init.d]# chkconfig --list tomcat  
tomcat          0:off   1:off   2:on    3:on    4:on    5:off   6:off  
```

现在我们来测试脚本！

启动Tomcat:

```bash
[root@srv6 ~]# service tomcat start  
Using CATALINA_BASE:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_HOME:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-8.0.23/temp  
Using JRE_HOME:        /usr/java/jdk1.8.0_25
Using CLASSPATH:       /usr/share/apache-tomcat-8.0.23/bin/bootstrap.jar:/usr/share/apache-tomcat-8.0.23/bin/tomcat-juli.jar
Tomcat started.
```

停止Tomcat:

```bash
[root@srv6 ~]# service tomcat stop  
Using CATALINA_BASE:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_HOME:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-8.0.23/temp  
Using JRE_HOME:        /usr/java/jdk1.8.0_25
Using CLASSPATH:       /usr/share/apache-tomcat-8.0.23/bin/bootstrap.jar:/usr/share/apache-tomcat-8.0.23/bin/tomcat-juli.jar
```

重启Tomcat(必须先启动):

```bash
[root@srv6 ~]# service tomcat restart  
Using CATALINA_BASE:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_HOME:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-8.0.23/temp  
Using JRE_HOME:        /usr/java/jdk1.8.0_25
Using CLASSPATH:       /usr/share/apache-tomcat-8.0.23/bin/bootstrap.jar:/usr/share/apache-tomcat-8.0.23/bin/tomcat-juli.jar  
Using CATALINA_BASE:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_HOME:   /usr/share/apache-tomcat-8.0.23
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-8.0.23/temp  
Using JRE_HOME:        /usr/java/jdk1.8.0_25
Using CLASSPATH:       /usr/share/apache-tomcat-8.0.23/bin/bootstrap.jar:/usr/share/apache-tomcat-8.0.23/bin/tomcat-juli.jar
Tomcat started.
```

我们应该检日志查看是否有错误

```bash
[root@srv6 init.d]# tail -f /usr/share/apache-tomcat-8.0.23/logs/catalina.out
```

我们现在可以访问Tomcat Manager页面:

`http://yourdomain.com:8080` 或者 `http://yourIPaddress:8080` 可以看到tomcat主页
![](https://biezhi.me/usr/uploads/2015/09/3030939225.jpg)

### 第四步：配置Tomcat Manager访问

出于安全原因,Tomcat manager没有用户或密码，默认为创建角色。在生产环境,最好是删除管理器应用程序。

设置角色,用户名和密码,我们需要配置tomcat/conf下面的`tomcat-user.xml`文件

默认情况下将`tomcat-users.xml`中的元素是被注释的

创建一个角色拥有如下权限：

+ manager-gui
+ manager-status
+ manager-jmx
+ manager-script
+ admin-gu
+ admin-script.

我们可以设置`manager gui`的角色,例如如下

```xml
<tomcat-users>  
  <role rolename="manager-gui"/>  
  <user username="tomcat" password="secret" roles="manager-gui"/>  
</tomcat-users>  
```

应该注意赋予多个角色,以免不安全。

### 第五步(可选)：使用JAVA_OPTS管理内存配置

正确配置堆内存取决于很多因素，为简单起见，我们将堆大小设置为相同的值128MB
添加JAVA_OPTS内存参数在我们的`Catalina.sh`文件。
下面编辑`Catalina.sh`文件设置堆大小

```bash
JAVA_OPTS="-Xms128m -Xmx128m"
```

我通常只是添加这个文件第二行:

```bash
#!/bin/sh  
JAVA_OPTS="-Xms128m -Xmx128m"   
# Licensed to the Apache Software Foundation (ASF) under one or more  
# contributor license agreements.  See the NOTICE file distributed with  
# this work for additional information regarding copyright ownership.  
# The ASF licenses this file to You under the Apache License, Version 2.0  
# (the "License"); you may not use this file except in compliance with  
# the License.  You may obtain a copy of the License at  
```

### 第六步(可选)：如何给指定的用户使用Tomcat

在上面的配置中我们使用ROOT用户运行Tomcat，处于安全原因，ROOT最好运行那些有必要的服务
当然没有规定必须这么做，但你最好谨慎点~

非ROOT用户运行Tomcat，需要做到以下几点：

1. 创建tomcat组:
 
  ```bash
  [root@srv6 ~]# groupadd tomcat  
  ```
  
2. 创建tomcat用户并将他加入到组
  
  ```bash
  [root@srv6 ~]# useradd -s /bin/bash -g tomcat tomcat  
  ```
  
  上面的写法将tomcat用户的家目录创建在`/home/tomcat`
  如果你想让主目录放在其他位置，可以使用-d参数
  
  ```bash
  [root@srv6 ~]# useradd -g tomcat -d /usr/share/apache-tomcat-8.0.23/tomcat tomcat 
  ```
  
  这样可以将tomcat用户的家目录设置为`/usr/share/apache-tomcat-8.0.23/tomcat`
  
3. 将tomcat目录的所有权给tomcat用户
  
  ```bash
  [root@srv6 ~]# chown -Rf tomcat.tomcat /usr/share/apache-tomcat-8.0.23/  
  ```
  
4. 调整tomcat的启动脚本，在新脚本中添加tomcat用户：
  
  ```bash
  #!/bin/bash  
  # description: Tomcat Start Stop Restart  
  # processname: tomcat  
  # chkconfig: 234 20 80  
  JAVA_HOME=/usr/java/jdk1.8.0_25  
  export JAVA_HOME  
  PATH=$JAVA_HOME/bin:$PATH  
  export PATH  
  CATALINA_HOME=/usr/share/apache-tomcat-8.0.23/bin  
    
  case $1 in  
  start)  
  /bin/su tomcat $CATALINA_HOME/startup.sh  
  ;;   
  stop)     
  /bin/su tomcat $CATALINA_HOME/shutdown.sh  
  ;;   
  restart)  
  /bin/su tomcat $CATALINA_HOME/shutdown.sh  
  /bin/su tomcat $CATALINA_HOME/startup.sh  
  ;;   
  esac      
  exit 0
  ```
  
### 第七部(可选):如何将tomcat运行在80端口

运行下面的服务端口1024是给root以外的用户,你可以添加到你的ipables:

```bash
[root@srv6 ~]# iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 8080    
[root@srv6 ~]# iptables -t nat -A PREROUTING -p udp -m udp --dport 80 -j REDIRECT --to-ports 8080    
```

重启iptables

```bash
service iptables restart
```

### 第八部(可选):运行Apache+Tomcat

在80端口上运行Tomcat,如果你有前面的Apache Tomcat,您可以使用使用Apache Tomcat的mod_proxy以及apj connector映射到vhost

当Tomcat是独立性能的改善,我仍然喜欢它前面的空间的原因。

在您的Apache配置,确保KeepAlive设置是`on`。 Apache调优,当然,本身是一个很大的话题……

*实例1: VHOST with mod_proxy:*

```xml
<VirtualHost *:80>  
    ServerAdmin admin@yourdomain.com  
    ServerName yourdomain.com  
    ServerAlias www.yourdomain.com  
  
  
    ProxyRequests Off  
    ProxyPreserveHost On  
    <Proxy *>  
       Order allow,deny  
       Allow from all  
    </Proxy>  
  
  
    ProxyPass / http://localhost:8080/  
    ProxyPassReverse / http://localhost:8080/  
  
  
    ErrorLog logs/yourdomain.com-error_log  
    CustomLog logs/yourdomain.com-access_log common  
  
</VirtualHost>  
```

*实例 2: VHOST with ajp connector and mod_proxy:*

```xml
<VirtualHost *:80>  
    ServerAdmin admin@yourdomain.com  
    ServerName yourdomain.com  
    ServerAlias www.yourdomain.com  
  
  
    ProxyRequests Off  
    ProxyPreserveHost On  
    <Proxy *>  
    Order allow,deny  
    Allow from all  
    </Proxy>  
  
    ProxyPass / ajp://localhost:8009/  
    ProxyPassReverse / ajp://localhost:8009/  
  
  
    ErrorLog logs/yourdomain.com-error_log  
    CustomLog logs/yourdomain.com-access_log common  
</VirtualHost>  
```

vhost在这两个例子,我们"映射"到Tomcat的根目录。

如果我们希望映射到应用程序如`yourdomain.com/myapp`,我们可以添加一些改写如下所示。

这将重写所有请求 `yourdomain.com` `yourdomain.com/myapp`

*实例 3: VHOST with rewrite:*

```xml
<VirtualHost *:80>  
    ServerAdmin admin@yourdomain.com  
    ServerName yourdomain.com  
    ServerAlias www.yourdomain.com  
  
  
    RewriteEngine On  
    RewriteRule ^/$ myapp/ [R=301]  
  
    ProxyRequests Off  
    ProxyPreserveHost On  
    <Proxy *>  
    Order allow,deny  
    Allow from all  
    </Proxy>  
  
    ProxyPass / ajp://localhost:8009/  
    ProxyPassReverse / ajp://localhost:8009/  
  
  
    ErrorLog logs/yourdomain.com-error_log  
    CustomLog logs/yourdomain.com-access_log common  
</VirtualHost>  
```
