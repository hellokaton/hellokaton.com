---
layout: post
title: CocoaPods安装遇到的坑
categories: ios
tags: cocoapods
---

CocoaPods是一个负责管理iOS项目中第三方开源代码的工具。如果你没听说过，也不想用，那就别往下看了。

<!-- more -->

1. CocoaPods的安装

需要用到ruby，Mac系统自带ruby，但如果不是最新的系统，最好更新一下。
ruby的软件源rubygems.org被墙了，所以先换一下源，命令行下依次执行3条命令

```sh
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l
```

然后升级gem

```sh
$ sudo gem update --system
```

完了就开始安装CocoaPods

```sh
$ sudo gem install cocoapods
$ pod setup
```

出现Setting up CocoaPods master repo，半天没有任何反应。原因无他，因为那堵墙阻挡了cocoapods.org。。。
gitcafe和oschina都是国内的服务器，可以用它们CocoaPods索引库的镜像：

```sh
$ pod repo remove master
$ pod repo add master https://gitcafe.com/akuandev/Specs.git
$ pod repo update
```

如果想用oschina的镜像也可以把第二条命令 换成 [http://git.oschina.net/akuandev/Specs.git](http://git.oschina.net/akuandev/Specs.git)即可

第二条命令执行的时候会比较耗时，这个时候要去把整个specs仓库clone一下，下载到 `~/.cocoapods`里；
cd  到该目录里，用du -sh *命令来查看文件大小，每隔一会看看，最终大小是190多M。


<!--more-->


2. CocoaPods的使用

（1）在终端shell中cd 来到你要管理的项目，运行：pod install 你的工程就多了个xworkspace文件夹，以后用这个打开工程

（2）添加第三方库
搜索一个开源库
```sh
$ pod search AFNetworking
```
在工程目录里建一个Podfile文件
```sh
$ vim Podfile
```
内容按这个格式来
```sh
platform :ios,'6.0'
pod 'RegexKitLite', '~> 4.0'
pod 'ASIHTTPRequest', '~> 1.8.2'
pod 'SDWebImage', '~> 3.7.1'
pod 'FMDB', '~> 2.3'
```


更多参考：
[CocoaPods一个Objective-C第三方库的管理利器](http://blog.csdn.net/totogo2010/article/details/8198694)
[CocoaPods进阶：本地包管理](http://ke.gitcafe.com/2013/04/18/advanced-cocoapods/)


3. CocoaPods的使用心得

（1）最近使用CocoaPods来添加第三方类库，无论是执行pod install还是pod update都卡在了Analyzing dependencies不动 原因在于当执行以上两个命令的时候会升级CocoaPods的spec仓库，加一个参数可以省略这一步，命令如下： 
```sh
pod install --verbose --no-repo-update pod update --verbose --no-repo-update
```
$ pod install只会按照Podfile的要求来请求类库，如果类库版本号有变化，那么将获取失败。但是 $ pod update会更新所有的类库，获取最新版本的类库。每次用$ pod update就行。

（2）安装一个xcode插件管理工具[http://alcatraz.io](http://alcatraz.io)，执行`curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh`  完了打开xcode->window->package manger 搜cocoapods安装，方便操作。


（3）工程在模拟器上编译报错，不支持i386，Cocoapods确实还不支持64位模拟器，解决办法：
[http://stackoverflow.com/questions/19213782/undefined-symbols-for-architecture-arm64](http://stackoverflow.com/questions/19213782/undefined-symbols-for-architecture-arm64)
其实就2条，
    1. build active architecture only 在debug的时候设置成YES，不要在release的时候用模拟器
    2. other linker flags 加一个 `$(inherited)`

（4）用到svn,git多人协作的话，Pods/这个文件夹不要上传，.../Pods/Pods.xcodeproj  ...Pods/Target Support Files/这些每次编译都会改动从而引起合并代码的时候冲突

更多坑可以看wiki，例如 [https://github.com/CocoaPods/CocoaPods/issues/2190](https://github.com/CocoaPods/CocoaPods/issues/2190)
其他坑各位可以补充。。。
