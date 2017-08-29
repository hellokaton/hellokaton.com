---
title: 聊聊TCP中的KeepAlive机制
date: 2017-08-29
category: ["王爵的技术小黑屋"]
tags: ["tcp", "keepalive"]
---

服务端的系统设置中经常会和底层协议打交道，我们有必要重温一下曾经那些“听过”却不熟悉的名词。
今天聊的话题是 `KeepAlive`，在实际应用中又是怎么使用的？

<!-- more -->

{% image /static/img/article/tcp-keepalive.png 650 340 wireshark抓包 %}

## 为什么有Keepalive？

<!--
在现实生活中我们都知道如果一个女生追了你很久你都没有答应，那么要不这个女生会放弃要么你是个`Gay`。
但最终都会有一个答案的，总不能妹子倾其一生在追你~ 而你却迟迟不应（yìng），那妹子的大好青春都被毁了。
正常一点的情况是妹子追了你一个月发现不来电，马上把目光转向
-->

大家都做过电梯吧，假设电梯来了你先进去，你朋友还没进来，过一段时间电梯门就会自动关闭，
你应该没遇到过哪个电梯会一直等你朋友来了才关门的。如果真是那样，那别的楼层的小姐姐们会炸了~

我们举个编程中的例子来解释下，我编写了一个服务端程序`S`和一个客户端程序`C`，客户端向服务端发送
一个消息：

{% image https://i.loli.net/2017/08/29/59a5633379ed8.png 350 250 客户端发送消息 %}

服务端收到消息后一看，瞧给你牛*的，然后没理客户端，傻狗客户端一直在等待，但是不知道是不是服务器挂掉了？
这时候`TCP`协议提出一个办法，当客户端端等待超过一定时间后自动给服务端发送一个空的报文，
如果对方回复了这个报文证明连接还存活着，如果对方没有报文返回且进行了多次尝试都是一样，
那么就认为连接已经丢失，客户端就没必要继续保持连接了。
如果没有这种机制就会有很多空闲的连接占用着系统资源。

> KeepAlive并不是TCP协议规范的一部分，但在几乎所有的TCP/IP协议栈（不管是Linux还是Windows）中，都实现了KeepAlive功能

[RFC1122#TCP Keep-Alives](https://tools.ietf.org/html/rfc1122#page-101)

## 如何设置它?

在设置之前我们先来看看`KeepAlive`都支持哪些设置项

1. KeepAlive默认情况下是关闭的，可以被上层应用开启和关闭
2. `tcp_keepalive_time`: KeepAlive的空闲时长，或者说每次正常发送心跳的周期，默认值为7200s（2小时）
3. `tcp_keepalive_intvl`: KeepAlive探测包的发送间隔，默认值为75s
4. `tcp_keepalive_probes`: 在tcp_keepalive_time之后，没有接收到对方确认，继续发送保活探测包次数，默认值为9（次）

我们讲讲在Linux操作系统和使用Java、C语言和Nginx中如何设置

### 在Linux内核设置

`KeepAlive`默认不是开启的，如果想使用`KeepAlive`，需要在你的应用中设置`SO_KEEPALIVE`才可以生效。

查看当前的配置：

```bash
cat /proc/sys/net/ipv4/tcp_keepalive_time
cat /proc/sys/net/ipv4/tcp_keepalive_intvl
cat /proc/sys/net/ipv4/tcp_keepalive_probes
```

在Linux中我们可以通过修改 `/etc/sysctl.conf` 的全局配置：

```bash
net.ipv4.tcp_keepalive_time=7200
net.ipv4.tcp_keepalive_intvl=75
net.ipv4.tcp_keepalive_probes=9
```

添加上面的配置后输入 `sysctl -p` 使其生效，你可以使用 `sysctl -a | grep keepalive` 命令来查看当前的默认配置

> 如果应用中已经设置`SO_KEEPALIVE`，程序不用重启，内核直接生效

### 使用Netty4设置

这里我们使用常用的Java网络框架Netty来设置，只需要在服务端设置即可：

```java
EventLoopGroup bossGroup   = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
            .channel(NioServerSocketChannel.class)
            .option(ChannelOption.SO_BACKLOG, 100)
            .childOption(ChannelOption.SO_KEEPALIVE, true)
            .handler(new LoggingHandler(LogLevel.INFO));

    // Start the server.
    ChannelFuture f = b.bind(8088).sync();
    // Wait until the server socket is closed.
    f.channel().closeFuture().sync();
} finally {
    // Shut down all event loops to terminate all threads.
    bossGroup.shutdownGracefully();
    workerGroup.shutdownGracefully();
}
```

这段代码来自经典的`echo`服务器，我们在`childOption`中开启了`SO_KEEPALIVE`。
Java程序只能做到设置`SO_KEEPALIVE`选项，其他配置项只能依赖于`sysctl`配置，系统进行读取。

### C语言设置

函数原型：

```c
#include <sys/socket.h>

int setsockopt(int socket, int level, int option_name,
      const void *option_value, socklen_t option_len);
```

我们在需要使能`Keepalive`的`socket`上面调用`setsockopt`函数便可以打开该`socket`上面的`keepalive`。

1. 第一个参数是要设置的套接字
1. 第二个参数是`SOL_SOCKET`
1. 第三个参数必须是`SO_KEEPALIVE`
1. 第四个参数必须是一个布尔整型值，0表示关闭，1表示打开
1. 最后一个参数是第四个参数值的大小。

调用例子：

```c
int socket(int domain, int type, int protocol)
{
  int (*libc_socket)(int, int, int);
  int s, optval;
  char *env;

  *(void **)(&libc_socket) = dlsym(RTLD_NEXT, "socket");
  if(dlerror()) {
    errno = EACCES;
    return -1;
  }

  if((s = (*libc_socket)(domain, type, protocol)) != -1) {
    if((domain == PF_INET) && (type == SOCK_STREAM)) {
      if(!(env = getenv("KEEPALIVE")) || strcasecmp(env, "off")) {
        optval = 1;
      } else {
        optval = 0;
      }
      if(!(env = getenv("KEEPALIVE")) || strcasecmp(env, "skip")) {
        setsockopt(s, SOL_SOCKET, SO_KEEPALIVE, &optval, sizeof(optval));
      }
#ifdef TCP_KEEPCNT
      if((env = getenv("KEEPCNT")) && ((optval = atoi(env)) >= 0)) {
        setsockopt(s, SOL_TCP, TCP_KEEPCNT, &optval, sizeof(optval));
      }
#endif
#ifdef TCP_KEEPIDLE
      if((env = getenv("KEEPIDLE")) && ((optval = atoi(env)) >= 0)) {
        setsockopt(s, SOL_TCP, TCP_KEEPIDLE, &optval, sizeof(optval));
      }
#endif
#ifdef TCP_KEEPINTVL
      if((env = getenv("KEEPINTVL")) && ((optval = atoi(env)) >= 0)) {
        setsockopt(s, SOL_TCP, TCP_KEEPINTVL, &optval, sizeof(optval));
      }
#endif
    }
  }

   return s;
}
```



代码摘取自`libkeepalive`源码，C语言可以设置更为详细的TCP内核参数

### 在Nginx中配置

在Nginx中配置TCP的`KeepAlive`非常简单，在`listen`指令下配置`so_keepalive`就可以了，具体配置

```bash
so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]
```

> this parameter (1.1.11) configures the “TCP keepalive” behavior for the listening socket. If this parameter is omitted then the operating system’s settings will be in effect for the socket. If it is set to the value “on”, the SO_KEEPALIVE option is turned on for the socket. If it is set to the value “off”, the SO_KEEPALIVE option is turned off for the socket. Some operating systems support setting of TCP keepalive parameters on a per-socket basis using the TCP_KEEPIDLE, TCP_KEEPINTVL, and TCP_KEEPCNT socket options. On such systems (currently, Linux 2.4+, NetBSD 5+, and FreeBSD 9.0-STABLE), they can be configured using the keepidle, keepintvl, and keepcnt parameters. One or two parameters may be omitted, in which case the system default setting for the corresponding socket option will be in effect.

**例子**

```bash
so_keepalive=30m::10
    will set the idle timeout (TCP_KEEPIDLE) to 30 minutes,
    leave the probe interval (TCP_KEEPINTVL) at its system default,
    and set the probes count (TCP_KEEPCNT) to 10 probes.
```

## 使用的场景

一般我们使用`KeepAlive`时会修改空闲时长，避免资源浪费，系统内核会为每一个TCP连接
建立一个保护记录，相对于应用层面效率更高。

常见的几种使用场景：

1. 检测挂掉的连接（导致连接挂掉的原因很多，如服务停止、网络波动、宕机、应用重启等）
2. 防止因为网络不活动而断连（使用NAT代理或者防火墙的时候，经常会出现这种问题）
3. TCP层面的心跳检测

`KeepAlive`通过定时发送探测包来探测连接的对端是否存活，
但通常也会许多在业务层面处理的，他们之间的特点：

- TCP自带的`KeepAlive`使用简单，发送的数据包相比应用层心跳检测包更小，仅提供检测连接功能
- 应用层心跳包不依赖于传输层协议，无论传输层协议是TCP还是UDP都可以用
- 应用层心跳包可以定制，可以应对更复杂的情况或传输一些额外信息
- `KeepAlive`仅代表连接保持着，而心跳包往往还代表客户端可正常工作

## 和Http中Keep-Alive的关系

1. HTTP协议的`Keep-Alive`意图在于连接复用，同一个连接上串行方式传递请求-响应数据
2. TCP的`KeepAlive`机制意图在于保活、心跳，检测连接错误

## 参考资料

- [Keepalive](https://en.wikipedia.org/wiki/Keepalive)
- [TCP Keepalive HOWTO](http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/)
- [随手记之TCP Keepalive笔记](http://www.blogjava.net/yongboy/archive/2015/04/14/424413.html)
- [理解TCP之Keepalive](http://www.firefoxbug.com/index.php/archives/2805/)