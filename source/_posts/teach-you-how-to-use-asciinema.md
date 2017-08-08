---
title: 教你使用asciinema录制命令行操作
date: 2017-08-08
category: ["linux"]
tags: ["asciinema", "terminal"]
---

[asciinema](https://asciinema.org/) 是一个在终端下非常棒的录制分享软件，基于文本的录屏工具，对终端输入输出进行捕捉，
然后以文本的形式来记录和回放！这使其拥有非常炫酷的特性：在 `播放` 过程中你随时可以暂停，
然后对“播放器”中的文本进行复制或者其它操作！实际效果可以点击下方的播放按钮查看。支持各个操作系统（哦。。没有windows）

<!-- more -->

{% image /static/img/article/asciinema-cover.png 600 350 asciinema %}

## 原理

将终端的操作记录成 `JSON` 格式，然后使用 `JavaScript` 解析，配合CSS展示，看起来像是视频播放器。
实际上就是文本，相比GIF和视频文件体积非常之小（时长2分50秒的录屏只有325KB），无需缓冲播放，
也可以方便的分享给别人或嵌入到网页中。

## 安装

**Mac**

```bash
brew install asciinema
```

**Pip安装**

```bash
sudo pip3 install asciinema
```

**Ubuntu**

```bash
sudo apt-add-repository ppa:zanchey/asciinema
sudo apt-get update
sudo apt-get install asciinema
```

**Arch Linux**

```bash
pacman -S asciinema
```

**Debian**

```bash
sudo apt-get install asciinema
```

**Fedora**

```bash
# Fedora < 22
sudo yum install asciinema
# Fedora >= 22
sudo dnf install asciinema
```

[更多安装姿势](https://asciinema.org/docs/installation)

## 举个栗子

安装ok后，我们来尝试录制一个试试？在你的终端输入 `asciinema rec` 回车后你会看到下面两行输出

```bash
~ Asciicast recording started.
~ Hit Ctrl-D or type "exit" to finish.
```

这表示录制已经开始，你可以按 `Ctrl+D` 或输入 `exit` 进行退出，下面是我录制的一个例子：

<script type="text/javascript" src="https://asciinema.org/a/132560.js" id="asciicast-132560" async></script>

退出后终端会输出

```bash
~ Asciicast recording finished.
~ Press <Enter> to upload, <Ctrl-C> to cancel.
```

按下回车即将你的录制上传到公共网站上，按下 `Ctrl+C` 即退出，本次操作不会保存。

## 还有什么？

**回放录制**

我们前面录制了一个，我可以使用 `asciinema play` 命令在本地回放刚才的录制操作

```bash
asciinema play https://asciinema.org/a/132560
```

回放本地的一个文件

```bash
asciinema play /path/132560.json
```

**嵌入播放**

本文中就是使用这种方式的

```bash
<script type="text/javascript" src="https://asciinema.org/a/132560.js"
id="asciicast-132560" async></script>
```

`markdown`的方式

```markdown
[![asciicast](https://asciinema.org/a/132560.png)](https://asciinema.org/a/132560)
```

**自由拷贝**

在播放记录时，可以自由地拷贝正在播放的记录中的命令，这一点上 `ttyrec` 和 `screen` 是无法相比的。
当你在观看别人的记录时，如果有一些非常炫酷的命令，当然会心痒难耐想要自己亲手试一试，这个时候这个特性实在是不要太赞！

**删除录制**

有时候不小心录制了一些隐私信息上去，不要担心，你可以在诸如 `https://asciinema.org/a/132556` 这个链接中登录进去删除它。

**配置文件**

在 `Asciinema` 的网站上，可以用自己的邮箱进行登录，在本地可以配置一下认证：

```bash
asciinema auth
vim ~/.config/asciinema/config
```

配置文件长下面这个样子

```bash
[api]
token = <your-api-token-here>
```

你也可以在这个文件中设置几个选项，这是一个所有可用的选项：

```bash
[api]
token = <your-api-token-here>
url = https://asciinema.example.com

[record]
command = /bin/bash -l
maxwait = 2
yes = true
quiet = true

[play]
maxwait = 1
```

`[api]` 中的选项与API位置和身份验证有关；
如果要让asciinema使用您自己的asciinema站点而不是默认的`asciinema.org`，可以设置`url`选项。
`API URL`也可以通过 `ASCIINEMA_API_URL` 环境变量传递。

`[record]`和`[play]`部分中的选项配合 `asciinema rec/asciinema play` 命令（请参阅其他参数）。

如果经常使用`-c`，`-w`或`-y`这些命令，推荐将其作为默认保存在配置文件中。

## 其他参数

```bash
☁  ~  asciinema -h
usage: asciinema [-h] [--version] {rec,play,upload,auth} ...

Record and share your terminal sessions, the right way.

positional arguments:
  {rec,play,upload,auth}
    rec                 Record terminal session(开始录制终端会话)
    play                Replay terminal session(播放终端会话)
    upload              Upload locally saved terminal session to asciinema.org(上传本地录制内容到asciinema)
    auth                Manage recordings on asciinema.org account(登录asciinema账号管理录制记录)

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

example usage:
  Record terminal and upload it to asciinema.org:
    asciinema rec
  Record terminal to local file:
    asciinema rec demo.json
  Record terminal and upload it to asciinema.org, specifying title:
    asciinema rec -t "My git tutorial"
  Record terminal to local file, "trimming" longer pauses to max 2.5 sec:
    asciinema rec -w 2.5 demo.json
  Replay terminal recording from local file:
    asciinema play demo.json
  Replay terminal recording hosted on asciinema.org:
    asciinema play https://asciinema.org/a/difqlgx86ym6emrmd8u62yqu8

For help on a specific command run:
  asciinema <command> -h
```

- `-t`：自定义名称，如 `asciinema rec -t "run first blade app"`
- `-w`：暂停时间最多多少秒，如 `asciinema rec -w 2.5 demo.json`，录制终端保存到本地，暂停时间最多2.5秒

[这里](https://asciinema.org/browse/featured) 有很多别人分享的有趣的录制。

