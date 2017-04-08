---
layout: post
title: centos下nginx启动、重启、关闭操作
categories: nginx
tags: centos nginx
---

## Nginx的启动

```sh
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf  
```

<!-- more -->

其中 `-c` 参数指定配置文件路径。

## Nginx的停止

Nginx支持以下几种信号控制：
- TERM, INT 快速关闭
- QUIT 从容关闭
- HUP 平滑重启
- USR1 重新打开日志文件，在切割文件时用处大
- USR2 平滑升级
- WINCH 从容关闭工作进程
- 
我们可以通过信号停止Nginx主进程，首先，我们需要通过 `ps -ef | grep` 命令获得master进程的PID，或者通过cat pid文件获得主进程号。下面是几个典型的停止语句：

```sh
#从容停止Nginx  
kill -QUIT master进程号  
#快速停止Nginx  
kill -TERM master进程号  
#强制停止Nginx  
kill -9 master进程号  
```

<!--more-->

## Nginx的重加载

如果改变了配置文件，想重启让其生效，同样可以通过发送系统信号给Nginx主进程，不过，在重启之前，要确认配置文件的语法是正确的，否则将不会加载新的配置项。
通过以下语句测试配置文件语法是否正确：

```sh
/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
```

其中 `-t` 表示测试，并不真正执行。
然后，通过以下命令重加载Nginx配置：

```sh
kill -HUP master进程号
```

执行上面命令之后，Nginx运行新的工作进程，旧工作进程继续为已有的连接服务，等所有旧的连接成功后，旧的工作进程才被关闭。

## Nginx的启动脚本

```sh
#!/bin/sh  
# chkconfig: 2345 85 15  
# description:Nginx Server  
  
NGINX_HOME=/usr/local/nginx  
NGINX_SBIN=$NGINX_HOME/sbin/nginx  
NGINX_CONF=$NGINX_HOME/conf/nginx.conf  
NGINX_PID=$NGINX_HOME/logs/nginx.pid  
  
NGINX_NAME="Nginx"  
  
. /etc/rc.d/init.d/functions  
  
if [ ! -f $NGINX_SBIN ]  
then  
    echo "$NGINX_NAME startup: $NGINX_SBIN not exists! "  
    exit  
fi  
  
start() {  
    $NGINX_SBIN -c $NGINX_CONF  
    ret=$?  
    if [ $ret -eq 0 ]; then  
        action $"Starting $NGINX_NAME: " /bin/true  
    else  
        action $"Starting $NGINX_NAME: " /bin/false  
    fi  
}  
  
stop() {  
    kill `cat $NGINX_PID`  
    ret=$?  
    if [ $ret -eq 0 ]; then  
        action $"Stopping $NGINX_NAME: " /bin/true  
    else  
        action $"Stopping $NGINX_NAME: " /bin/false  
    fi  
}  
  
restart() {  
    stop  
    start  
}  
  
check() {  
    $NGINX_SBIN -c $NGINX_CONF -t  
}  
  
  
reload() {  
    kill -HUP `cat $NGINX_PID` && echo "reload success!"  
}  
  
relog() {  
    kill -USR1 `cat $NGINX_PID` && echo "relog success!"  
}  
  
case "$1" in  
    start)  
        start  
        ;;  
    stop)  
        stop  
        ;;  
    restart)  
        restart  
        ;;  
    check|chk)  
        check  
        ;;  
    status)  
        status -p $NGINX_PID  
        ;;  
    reload)  
        reload  
        ;;  
    relog)  
        relog  
        ;;  
    *)  
        echo $"Usage: $0 {start|stop|restart|reload|status|check|relog}"  
        exit 1  
esac
```

上面是nginx的启动脚本，只要把它拷贝至 `/etc/init.d` 目录下，就可以通过 `service nginx start` 等目录操作nginx。

除了上面介绍的直接发信号给Nginx主进程的方法之外，我们还可以通过nginx -s命令：

- stop — fast shutdown
- quit — graceful shutdown
- reload — reloading the configuration file
- reopen — reopening the log files

