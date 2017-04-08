---
layout: post
title: centos安装pptp、vpn
categories: linux
tags: centos vpn
---

最近换了台IOS的手机，以前安卓用Shadowsocks（影梭）即可，刚好手里有台国外的VPS在挂网站，
顺便搭建一个pptp的服务在IOS上使用，pptp的搭建比openvpn容易多了！

## 安装步骤

1. 检查环境
2. 安装ppp和iptables
3. 修改配置文件
4. 启动pptp vpn服务和iptables

<!-- more -->

### 检查环境

先检查vps是否满足配置pptp vpn的环境。因为有的openvz的vps被母鸡给禁用了。其实，你在配置前最好向vps的客服发TK，可能客服会帮你开通vpn或者客服那里会给你他们自己定制的vpn一键安装包也有可能，我的机器是Linode KVM架构的。

服务器版本：CentOS release 6.5 (Final)
内核版本：

这里说一下，如果你的linux内核版本 等于或高于 2.6.15 ，内核集成了MPPE。可以用下面命令进行测试内核是否支持

```sh
modprobe ppp-compress-18 && echo ok
```

返回 `ok` 说明测试通过。但是返回报错 `FATAL: Module ppp_mppe not found.` ，也不能说不支持，因为modprobe命令是去 `/lib/modules/`uname -r` 找模块，但是很多时候，这个目录下是空的。所以这个命令没什么太大用处。
于是有人又提出一个命令，通过查看内核编译的配置文件config.gz：

```sh
zgrep MPPE /proc/config.gz
```

返回CONFIG_PPP_MPPE=y 或 =m说明内核已经编译了MPPE，通过测试。但是呢，这个命令其实也没什么用，因为有的vps空间商会不会备份config.gz文件。所以，config.gz文件都没有，这命令也是废了。
所以，最后建议直接使用下面的指令：

```sh
cat /dev/net/tun
```

如果这条指令显示结果为下面的文本，则表明通过：

```sh
cat: /dev/net/tun: File descriptor in bad state
```

上述任意一个命令测试通过，就能安装pptp。否则就只能考虑openvpn。

确认自己的vps能够支持pptp vpn 或其他类型的vpn。最好的方法是直接问vps空间商，因为没有人比他们更清楚了。没准人家还会提供vpn一键安装包呢！！！
有部分的vps需要发tk，让vps空间商的技术客服为你的VPS打开TUN/TAP/PPP功能了，而有部分vps控制面板上提供打开TUN/TAP/PPP功能的按钮，自己就能手动开启。

***Centos 6.4内核版本在2.6.15以上，都默认集成了MPPE和PPP，因此下面检查可以忽略：***

```sh
rpm -q ppp //查询当前系统的ppp是否默认集成了，以及ppp的版本
```

检查PPP是否支持MPPE
用以下命令检查PPP是否支持MPPE：

```sh
strings '/usr/sbin/pppd' | grep -i mppe | wc –lines
```

如果以上命令输出为"0"则表示不支持；输出为"30"或更大的数字就表示支持，MPPE（Microsoft Point to Point Encryption，微软点对点加密）。

### 安装ppp和iptables

接着是安装配置pptp vpn的相关软件，安装ppp和iptables。

#### 1. 安装ppp和iptables

PPTPD要求Linux内核支持mppe，一般来说CentOS安装时已经包含了,centos默认安装了iptables和ppp

```sh
yum install -y perl ppp iptables
```

#### 2.安装pptpd

刚才用了yum安装了ppp,但是这里有个问题，几乎大部分的人都会在这里遇到ppp和pptpd不兼容的错误。
因为yum安装ppp，总是安装最新版本的ppp，而由于安装的ppp的版本不同，那么就需要安装对应版本的pptpd才行。

安装pptpd的方法是直接用yum安装，让电脑自动选择对应的版本：

**先加入yum源：**

```sh
rpm -Uvh http://poptop.sourceforge.net/yum/stable/rhel6/pptp-release-current.noarch.rpm
```

然后用yum安装pptpd：

```sh
yum install pptpd
```

64位安装的时候如果出现：

```sh
warning: pptpd-1.3.4-2.rhel5.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 862acc42: NOKEY
error: Failed dependencies:
ppp = 2.4.4 is needed by pptpd-1.3.4-2.rhel5.x86_64
原因是pptpd与PPP不兼容，那么，此时用#yum list installed ppp 命令查看ppp版本，极有可能ppp是2.4.5版本的。所以，我们要下载pptp 1.4.0版本才行，而且这里是64位的系统。下载pptpd-1.4.0-1.el6.x86_64.rpm安装即可。这就是我说的出现版本不兼容的问题，当ppp版本和pptpd版本不兼容时候，就会出现类似的错误。
这里我分享下pptpd 下载地址；
64位pptpd-1.4.0-1.el6.x86_64.rpm的下载地址：http://www.pipipan.com/file/18457333
32位pptpd-1.4.0-1.el6.i686.rpm版本下载地址：http://www.400gb.com/file/54124192
看到有人建议用–nodeps –force 这个参数，我个人不建议，这个参数可能以后会出现奇怪的问题，但是如果实在不行，你就用吧
```

### 修改配置文件

配置安装好后的pptp软件，这个不像windows那样，安装的过程就是配置的过程。linux的要安装完之后，修改配置文件，才算是完成配置。

1. 配置文件 `/etc/ppp/options.pptpd`

```sh
cp /etc/ppp/options.pptpd /etc/ppp/options.pptpd.bak
```

```sh
vim /etc/ppp/options.pptpd
```

> 解析：我还建议是在原配置文件上添加内容来配置pptp ，省的不必要的麻烦和问题

将如下内容添加到到options.pptpd中：

```sh
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

然后保存这个文件。

> 解析：ms-dns 8.8.8.8， ms-dns 8.8.4.4是使用google的dns服务器。

2. 配置文件 `/etc/ppp/chap-secrets`

```sh
cp /etc/ppp/chap-secrets /etc/ppp/chap-secrets.bak
```

```sh
vim /etc/ppp/chap-secrets
```

chap-secrets添加如下内容：

```sh
# 用户名：myusername 密码：mypassword
myusername pptpd mypassword *
```

3. 配置文件 `/etc/pptpd.conf`

```sh
cp /etc/pptpd.conf /etc/pptpd.conf.bak
```

```sh
vim /etc/pptpd.conf
```

**添加下面两行：**

```sh
localip 192.168.9.1
remoteip 192.168.9.11-30 //表示vpn客户端获得ip的范围
```

> 关键点：pptpd.conf这个配置文件必须保证最后是以空行结尾才行，否则会导致启动pptpd服务时，出现“Starting pptpd:”，一直卡着不动的问题，无法启动服务，切记呀！（相关文档可以查看：[Starting pptpd: 运行不下去的原因](http://www.dabu.info/starting-pptpd-does-not-run-down-turn.html)）

4. 配置文件 `/etc/sysctl.conf`

修改内核设置，使其支持转发

```sh
vim /etc/sysctl.conf
```

将 `net.ipv4.ip_forward = 0` 改成 `net.ipv4.ip_forward = 1`

保存修改后的文件执行 `/sbin/sysctl -p`


### 启动pptp vpn服务和iptables

启动pptp vpn 服务。此时，就是检验你能够vpn拨号成功，如果你拨号成功了，说明你的pptp vpn的安装配置就算真正的完成了。但是此时只能登录vpn，却不能用来上网。

开启内核和iptables的转发功能。这个步骤是为了让你连上vpn之后，能够上网，上那些yourporn，youtube之类的。这步是最关键的，很多人能成功拨号，登录vpn，但是却不能上网就是因为这个步骤没做好。这步骤完成了，你就可以尽情去国外的网站访问了。

```sh
/sbin/service pptpd start
```

或者

```sh
service pptpd start
```

经过前面步骤，我们的VPN已经可以拨号登录了，但是还不能访问任何网页。最后一步就是添加iptables转发规则了，输入下面的指令：

*启动iptables和nat转发功能，很关键：*

```sh
/sbin/service iptables start //启动iptables
/sbin/iptables -t nat -A POSTROUTING -o eth0 -s 192.168.9.0/24 -j MASQUERADE
/etc/init.d/iptables save //保存iptables的转发规则
/sbin/service iptables restart //重新启动iptables
```

**最后一步：重启pptp vpn**

```sh
service pptpd restart
```

出现 `Setting chains to policy ACCEPT: security raw nat`

解决方法：

```sh
vim /etc/init.d/iptables
```

```sh
    echo -n $"${IPTABLES}: Setting chains to policy $policy: "
    ret=0
    for i in $tables; do
        echo -n "$i "
        case "$i" in
+           security)
+               $IPTABLES -t filter -P INPUT $policy \
+                   && $IPTABLES -t filter -P OUTPUT $policy \
+                   && $IPTABLES -t filter -P FORWARD $policy \
+                   || let ret+=1
+               ;;
            raw)
                $IPTABLES -t raw -P PREROUTING $policy \
                    && $IPTABLES -t raw -P OUTPUT $policy \
                    || let ret+=1
                ;;
```

带有+号的是新增的脚本，修改后保存

客户端如何拨号登陆vpn，我就不写了，大家可以自行google，因为系统那么多，我不可能xp，win7，centos，mac之类的每个都写，何况网上一大堆，只要你pptp vpn服务器搭建好了，客户端登陆的选择就是简单的事。如果这个也不知道，那我就没法了，自己动手吧。


**多余的步骤：设置pptp vpn 开机启动**

有的人懒的重启后手动开启服务，所以下面我再补上开机自动启动pptp vpn 和 iptables的命令

```sh
chkconfig pptpd on //开机启动pptp vpn服务
chkconfig iptables on //开机启动iptables
```

上2张图：

![配置PPTP](http://i5.tietuku.com/59b2bcddf62d5114.png)

![IOS访问GoogleChrome][1]


  [1]: http://i5.tietuku.com/36897cfee685c3d8.png