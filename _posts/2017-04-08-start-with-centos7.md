---
layout:     post
title:      "开始进击CentOS7"
category:    linux
tags: ["linux"]
---

CentOS 是一个工业标准的 Linux 发行版，是红帽企业版 Linux 的衍生版本。你安装完后马上就可以使用，但是为了更好地使用你的系统，你需要进行一些升级、安装新的软件包、配置特定服务和应用程序等操作。我之前一直使用的是Centos6系列操作系统作为服务器主力，由于 [微服务架构](http://microservices.io/patterns/microservices.html) 的普及和容器化的流行，有必要开始使用 `CentOS7` 了，这篇文章记录一下我从安装和之前使用 `CentOS6` 的一些不同之处，作为记录。

<!-- more -->

## 优势

CentOS7 兼容上游供应商的在分配策略，通过安全更新并获得全行业的支持。事实上，CentOS是唯一与流行的cPanel w虚拟主机控制面板兼容的操作系统。
CentOS 7是一个非常稳定的操作系统，当正确配置好，并在高质量的硬件上运行时，很少有问题，减少了崩溃和错误的风险。
您可以通过计算机配置共同工作，使用一组共享公共文件系统的服务器，并通过提高可用性应用程序，来提高性能和负载平衡资源。
CentOS 7使用者获得更新后的企业级安全功能，其中包括强大的防火墙和SELinux策略机制。
与市场上的其他发行版本，CentOS 7不需要频繁的版本更新，它具有卓越的长期稳定性，很少出现错误或者安全漏洞。

内核更新到3.10.0

- 支持Linux容器
- LVM快照支持ext4和XFS
- 转用systemd、firewalld和GRUB2
- XFS作为缺省文件系统
- 支持PTPv2
- 支持40G 以太网卡
- 在兼容的硬件上支持以UEFI安全启动模式安装

这其中最令人瞩目的新特性就是支持 `Docker` 技术。作为目前流行的应用虚拟化技术之一，`Docker` 能够将应用程序与系统完全隔离，让其在系统之间实现迁移而不需要停机，提高了应用程序的移动性和灵活性。`CentOS7` 在内核层面支持 `Docker` 容器技术，可以提高 `Docker` 稳定性和可靠性。


## 安装

1. 下载centos7 [镜像](http://isoredirect.centos.org/centos/7/isos/x86_64/)
2. 我在虚拟机下进行安装的，具体的操作就不详述了，和之前是类似的，可以看看我之前的 [教程](https://github.com/biezhi/java-bible/blob/master/learn_server/README.md)

相对于CentOS6而言其实更简单了，而且启动更快。只要按照安装引导一步一步执行即可，这里值得一提的是磁盘默认是基于LVM的（自定义分区的时候可以看到）。当然，我们安装的时候可以选用自动分区。

有一个分区方案：

1. `/boot`:200M
2. `/swap`:2048M
3. `/usr`:20G
4. `/`:20G
5. `/data`:剩余

关于SWAP分区大小：8G以下给当前内存的2倍即可，8G以上一律给16G即可（再给大了也是浪费磁盘空间）。

### 启动模式

我们熟悉的CentOS 6中，系统有7个运行级别。

- `0`: 关机
- `1`: 单用户
- `2`: 多用户 – NO NFS
- `3`: 多用户
- `4`: 保留
- `5`: x11图形界面
- `6`: 重启

而在7中，这7个级别转化为4个 `target`，分别如下：

- `Graphical.target`：多人模式，支持图形和命令行两种登录，对应6中的3和5。
- `Multi-User.target`: 多人模式，只支持命令行登录，对应6中的3。
- `Rescue.target`: 单人模式,对应于之前的1级别
- `Emergency.target`: 单人模式，不过系统启动后根目录是只读模式，

### Emergency.target

我们如果忘记系统的root密码，可以进入该模式进行重置。`CentOS7` 中引入了 `Grub 2`（内核条目选择界面按E编辑，为了不错过该界面，我们启动后可以按几下上下方向键），我们可以发现与之前的 `grub` 配置有很大区别，多了很多设置内容。我们这里要进入 `emergency.target`，只需要在 `"linux16"` 所在行的最后添加 `"rd.break"`。这里要特别说明一点，如果之前安装系统选择了非英文的系统语言，此行原本最后一个设置 `LANG` 建议改回 `"en_US.UTF-8"`, 否则进入系统命令窗口执行命令可能会出现乱码（当然，我们可以进入系统再修改）。按 `ctrl+x`后，我们就进入了 `emergency.target` 模式中。

目前，我们的系统是只读的，我们需要重新挂载一下即可。如下：

```bash
mount -o remount,rw /sysroot/
chroot /sysroot/ #切换到原始系统
touch /.autorelabel #这句是让selinux生效
echo $LANG
LANG=en
passwd #修改密码
exit #退出
init 6 #重启系统
```

没错，为了先后兼容，7中仍然支持 `init + 启动级别的指令`。这意味着我们可以用 `init 0` 关机、`init 6` 重启那。正统的相关指令为 `systemctl` 。进入系统是，我们不难感受到：CentOS7启动速度还是比较快，这得益于之后要说明的systemd服务。

### Rescue.target: 救援模式

在CentOS 7中，救援模式的进入和6倒区别不大。步骤如下：

1. BIOS设置光驱启动。
2. CentOS 7安装选项页面：我们选择troubelshooting。
3. 在troubleshooting选项界面选择”rescue a CentOS system”即可。
4. 选择需要进入的救援模式(可写、只读)
5. 进入所选择的救援模式，执行指令“chroot /mnt/sysimage”，挂载原来的系统

`chroot /mnt/sysimage`

## 安装后要做的

### 添加第三方源

CentOS 由于很追求稳定性，所以官方源中自带的软件不多，因而需要一些第三方源，比如 EPEL、ATrpms、ELRepo、Nux Dextop、RepoForge 等。根据上面提到的软件安装原则，为了尽 可能保证系统的稳定性，此处大型第三方源只添加 EPEL 源、Nux Dextop 和 ELRepo 源。

**EPEL**

EPEL 即 `Extra Packages for Enterprise Linux`， 为 CentOS 提供了额外的 10000 多个软件包，而且在不替换系统组件方面下了很多功夫，因而可以放心使用。

`sudo yum install epel-release`

执行完该命令后，在 `/etc/yum.repos.d` 目录下会多一个 `epel.repo` 文件。

**阿里云源**

1、_备份_

```bash
yum -y install wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2、下载新的`CentOS-Base.repo` 到`/etc/yum.repos.d/`

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

3、运行`yum makecache`生成缓存

### 安装 yum-axelget

[yum-axelget](https://dl.fedoraproject.org/pub/epel/7/x86_64/repoview/yum-axelget.html) 是 EPEL 提供的一个 yum 插件。使用该插件后用 yum 安装软件时可以并行下载，大大提高了软件的下载速度，减少了下载的等待时间:

`sudo yum install yum-axelget`

安装该插件的同时会安装另一个软件 axel。axel 是一个并行下载工具，在下载 http、ftp 等简单协议的文件时非常好用。

### 第一次全面更新

在进一步操作之前，先把已经安装的软件包都升级到最新版:

`sudo yum update`

要更新的软件包有些多，可能需要一段时间。不过有了 yum-axelget 插件，速度已经快了很多啦。

### 重启

第一次全面更新完之后建议重启。

### 基础开发环境

**GCC 系列**

```bash
sudo yum install gcc                     # C 编译器
sudo yum install gcc-c++                 # C++ 编译器
sudo yum install gcc-gfortran            # Fortran 编译器
sudo yum install compat-gcc-44           # 兼容 gcc 4.4
sudo yum install compat-gcc-44-c++       # 兼容 gcc-c++ 4.4
sudo yum install compat-gcc-44-gfortran  # 兼容 gcc-fortran 4.4
sudo yum install compat-libf2c-34        # g77 3.4.x 兼容库
```

### 软件开发辅助工具

```bash
sudo yum install make
sudo yum install cmake   # Cmake
sudo yum install git     # 版本控制
```

## 和之前的区别

| 命令 | CentOS6 | CentOS7 |
| ------| ------ | ------ |
| `ifconfig` | 有 | 有 `yum insall -y net-tools` |
| `rouet` | 有 | 有 `yum insall -y net-tools` |
| `ntpd`服务和`ntpdate`命令 | 有 | 有 `yum insall -y ntp ntpdate` |
| `cat /etc/issue` | 有版本号 | 无信息，只能查看`cat /etc/redhat-release` |
| `setup` | 能更改网络配置 | `setup`去除了防火墙和网路配置，通过安装 `yum -y install NetworkManager-tui `<br/>`nmtui` 命令取代了`setup`中的网络配置 |
| 时区和时间设置 | `/etc/sysconfig/clock`等文件 | `timedatectl set-timezone Asia/Shanghai`<br/>`timedatectl status` |
| 语言等设置 | `locale -a` | `localectl status` |
| 服务管理 | `chkconfig` `/etc/init.d/`服务 | `systemctl` |
| `python` | 默认2.6 | 默认2.7 |
| `kernel` | 默认2.6 | 默认3.10 |
| 网卡 | eth0 | 成为了可预见性的命名规则 |
| 文件系统 | `ext4` | `xfs` |
| `dig nslookup` | 有 | 有 `yum install bind-utils -y` |
| 主机名 | `cat /etc/sysconfig/network` | `cat /etc/hostname` |
| 服务的管理和控制 | `sysvinit` | `systemctl`是最主要的工具，它融合
`service` 和 `chkconfig` 的功能于一体。 |
| 防火墙 | `iptables` |  被`firewalld`取代 |
| 启动级别 | `/etc/inittab` | 不在使用了 |
| 内核参数配置 | `/etc/sysctl.conf` | `/usr/lib/sysctl.d/00-system.conf` 和 `/etc/sysctl.d/<name>.conf` |
| init关机重启命令 | init 0 关机 | init 0 关机 |
| 切换等级 | 切回单用户模式 `init 0` | `init 0`<br/>`systemctl emergency`<br/>`systemctl isolate runlevel1.target` |

### IP设置

- 网卡名称不再是`eth0`、`eth1`,而是eno+8位数字。
- `dhclient`先自动获取IP
- 默认不再支持`ifconfig`命令，而是使用`ip addr`查看。不过我们可以自行安装`net-tools`包（`yum install -y net-tools`）。
- 重启网络命令：`systemctl restart network.service`

### 主机名设置

```bash
hostname #5\6\7都是使用该命令查看
hostname putty.biz #只在当前shell中有效
hostnamectl set-hostname putty.biz #设置主机名，直接更改配置文件/etc/hostname中的设置。
hostname status #这个命令不但包括主机名，还有很多其他信息，例如内核、系统版本等
```

### 加强版的TAG命令补全

在CentOS中，我们可以让参数也被自动补全。不过，最小化安装是不支持的，我们需要额外手动安装一个RPM包: `bash-completion`

```bash
yum install -y bash-completion
source /etc/profile
hostnamectl set- #我们试试TAB补全吧。。。
```

### 服务相关systemd

在CentOS 6中，我们熟悉的服务相关命令chkconfig、service等不再被提供，取而代之的是systemd（对应的命令是systemctl），使用概况如下：

```bash
systemctl enable httpd.service #自启动某服务
systemctl disable httpd.service #禁用开机启动
systemctl status httpd.service #查看服务状态
systemctl list-units --type=service #查看所有服务
systemctl start httpd.service
systemctl stop httpd.service
systemctl restart httpd.service
systemctl is-enabled httpd #查看httpd服务是否开机启动
```

而对于启动脚本的存放位置，也不再是`/etc/init.d/`（这个目录也是存在的），而是`/usr/lib/systemd/system`

```bash
ls /usr/lib/systemd/system
```

`systemd`中的服务是可以并行启动的，只要它们之间不存在依赖关系(支持自动检测服务依赖的服务)，这也是为什么`CENTOS7`开机启动要更快的原因之一。 不过`systemd`机制相对6中的`init.d`机制（顺序启动））要复杂得多。正所谓舍得嘛！

### 防火墙firewalld

CentOS 6中的防火墙程序`netfilter`也被替换为`firewalld`。不过，为了保持先后兼容性，我们仍然可以使用`netfilter`，方法如下：

```bash
systemctl stop firewalld
systemctl disable firewalld
yum installed -y iptables-services
systemctl enable iptables
systemctl start iptables
```

之后，我们就可以按照`Netfilter`那一套去操作。

