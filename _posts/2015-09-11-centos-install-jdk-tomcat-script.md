---
layout: post
title: centos上一键安装jdk、tomcat脚本
categories: linux
tags: centos jdk tomcat
---

## 下载jdk

Oracle不允许直接从他们的服务器下载jdk，看这里 [http://www.oracle.com/technetwork/java/javase/terms/license/index.html]

<!-- more -->


所以如果你尝试这样：

```bash
wget "http://download.oracle.com/otn-pub/java/jdk/7u4-b20/jdk-7u4-linux-x64.tar.gz"
```

你将得到一个许可条款的页面，幸运的是,你需要一个cookie可以绕过这个：

```bash
Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie
```

所以,如果你想下载64位的Linux(例如jdk7u4使用wget,Ubuntu),您可以使用:

```bash
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.tar.gz"
```

仅供参考,以下是JDK8的下载项目，其他的可以在oracle官网找到

### JDK 8u60

+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-i586.tar.gz
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.tar.gz
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-solaris-x64.tar.gz
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-windows-i586.exe
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-windows-x64.exe
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-macosx-x64.dmg
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-i586.rpm
+ http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.rpm

## 安装脚本

```bash
#!/bin/bash

# jdk install
# 请将下载的jdk-xxx-linux-xxx.tar.gz包与此脚本放置到同一目录
# 授予此脚本可执行权限(chmod +x install_jdk.sh)
# 在终端执行此脚本开始安装(./文件名)
# 注意：不可有多个版本的jdk包！
#      为了使配置的环境变量生效，安装完成后你应该重新登陆。

echo "Please select you want to install the JDK version?"
select jdk_version in "JDK1.7" "JDK1.8" ; do
  break;
done

jvmpath=/usr/local/java/
# 不存在
if [ ! -d "$jvmpath" ]; then
    echo "正在创建$jvmpath目录"
    sudo mkdir $jvmpath
    echo "目录$jvmpath创建成功"
fi

jdkfile=$(ls | grep jdk-*-linux-*.gz)
jdkdirname=""

if [ "$jdk_version" = "JDK1.7" ]; then
    jdkdirname="jdk1.7.0_75"
else
    jdkdirname="jdk1.8.0_20"
fi

os_version=`uname -a`
echo $os_version
architecture="64"
echo "$os_version" | grep -q "$architecture"
if [ $? -eq 0 ]
then
    echo "您正在使用64位操作系统，为您选择64位JDK"
    # 不存在即去外网下载jdk文件
    if [ ! -f "$jdkfile" ]; then
        if [ "$jdk_version" = "JDK1.7" ]; then
            wget http://7vil1r.com1.z0.glb.clouddn.com/jdk-7u75-linux-x64.tar.gz
        else
            wget http://7vil1r.com1.z0.glb.clouddn.com/jdk-8u20-linux-x64.gz
        fi
    fi  
    #sudo chown -R jiangxin:jiangxin /usr/lib/jvm/jdk1.7.0_75
else
    echo "您正在使用32位操作系统，为您选择32位JDK"
    # 不存在即去外网下载jdk文件
    if [ ! -f "$jdkfile" ]; then
        if [ "$jdk_version" = "JDK1.7" ]; then
            wget http://7vil1r.com1.z0.glb.clouddn.com/jdk-7u75-linux-i586.gz
        else
            wget http://7vil1r.com1.z0.glb.clouddn.com/jdk-8u20-linux-i586.gz
        fi
    fi
fi

jdkfile=$(ls | grep jdk-*-linux-*.gz)

if [ -f "$jdkfile" ]; then

    sudo tar -zxvf $jdkfile -C /usr/local/java/

    echo "安装JDK成功"

    echo "配置环境变量"
    # touch environment  
    # echo "PATH=\"$PATH:/usr/lib/jvm/$jdkdirname/bin\"" >> environment
    # echo "JAVA_HOME=/usr/lib/jvm/$jdkdirname" >> environment
    # echo "CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar" >> environment
    # sudo mv /etc/environment /etc/environment.backup.java
    # sudo mv environment /etc
    # source /etc/environment

    mv ~/.bashrc ~/.bashrc.backup.java
    cat ~/.bashrc.backup.java >> ~/.bashrc
    echo "PATH=\"$PATH:$jvmpath/$jdkdirname/bin\"" >> ~/.bashrc
    echo "JAVA_HOME=$jvmpath/$jdkdirname" >> ~/.bashrc
    echo "CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar" >> ~/.bashrc
    source ~/.bashrc
    echo "配置环境成功"

    # 如果有多个java版本需要进行以下配置（包括openjdk）
    echo "设置默认jdk"
    sudo update-alternatives --install /usr/bin/java java $jvmpath/$jdkdirname/bin/java 300
    sudo update-alternatives --install /usr/bin/javac javac $jvmpath/$jdkdirname/bin/javac 300
    sudo update-alternatives --config java
    # echo "设置默认jdk成功"

    echo "测试是否安装成功"
    java -version
    echo "安装成功"

fi
```

以上脚本中下载JDK的方式根据自己需求修改即可。
## 卸载JDK脚本

```bash
#!/bin/bash

echo "正在删除相关文件"
sudo rm -rf /usr/local/java/
wait
echo "删除相关文件成功"

echo "恢复配置文件"
# sudo rm -f /etc/environment
# sudo mv /etc/environment.backup.java /etc/environment
sudo rm /usr/bin/java /usr/bin/javac
sudo rm /etc/alternatives/java /etc/alternatives/javac
mv ~/.bashrc.backup.java ~/.bashrc
echo "恢复配置文件成功"
```

## 安装tomcat脚本

```bash
#!/bin/bash


echo "Please select you want to install the Tomcat version?"
select tomcat_version in "Tomcat7x" "Tomcat8x" ; do
  break;
done

tomcatpath=/usr/local/webserver/
# 不存在
if [ ! -d "$tomcatpath" ]; then
    echo "正在创建$tomcatpath目录"
    sudo mkdir $tomcatpath
    echo "目录$tomcatpath创建成功"
fi

tomcatfile=$(ls | grep apache-tomcat-*.gz)

tomcatname=""

if [ "$tomcat_version" = "Tomcat7x" ]; then
    tomcatname="tomcat7"
else
    tomcatname="tomcat8"
fi

# 不存在即去外网下载jdk文件
if [ ! -f "$tomcatfile" ]; then
    if [ "$tomcat_version" = "Tomcat7x" ]; then
        wget http://apache.cs.utah.edu/tomcat/tomcat-7/v7.0.63/bin/apache-tomcat-7.0.63.tar.gz
    else
        wget http://mirror.tcpdiag.net/apache/tomcat/tomcat-8/v8.0.23/bin/apache-tomcat-8.0.23.tar.gz
    fi
fi  
    
tomcatfile=$(ls | grep apache-tomcat-*.gz)

if [ -f "$tomcatfile" ]; then

    sudo tar -zxvf $tomcatfile -C $tomcatpath
    
    sudo mv $tomcatpath$tomcatfile $tomcatname
        
    echo "安装Tomcat成功"
    
    echo "配置环境变量"
    
    mv ~/.bashrc ~/.bashrc.backup.tomcat
    cat ~/.bashrc.backup.tomcat >> ~/.bashrc
    
    #echo "PATH=\"$PATH:$tomcatpath$tomcatname\"" >> ~/.bashrc
    echo "TOMCAT_HOME=$tomcatpath$tomcatname" >> ~/.bashrc
    echo "CATALINA_HOME=$tomcatpath$tomcatname" >> ~/.bashrc
    echo "export PATH=\"$PATH:$tomcatpath$tomcatname\"" >> ~/.bashrc
    
    source ~/.bashrc
    
    echo "配置环境成功"
    
    echo "安装成功"

fi
```

## 卸载tomcat

```bash
#!/bin/bash

echo "正在删除相关文件"
sudo rm -rf /usr/local/webserver/
wait
echo "删除相关文件成功"

echo "恢复配置文件"
mv ~/.bashrc.backup.tomcat ~/.bashrc
echo "恢复配置文件成功"
```