---
layout: post
title: unsupported major.minor version 解决方法
categories: java
tags: major
---

何谓 major.minor，且又居身于何处呢？先感性认识并找到 major.minor 来。顺便写一段 代码，然后用 JDK 1.5 的编译器编译成class，用UltraEdit或者其他能打开非十进制文件的软件打开此class，见下图:    
    
![1.jpg](https://dn-biezhi.qbox.me/2015/10/1221350593.jpg)

从上图中我们看出来了什么是 major.minor version 了，它相当于一个软件的主次版本号，只是在这里是标识的一个 Java Class 的主版本号和次版本号，同时我们看到 minor_version 为 0x0000，major_version 为 0x0031，转换为十制数分别为0 和 49，即 major.minor 就是 49.0 了。

<!-- more -->

现在不妨从 JDK 1.1 到 JDK 1.7 编译器编译出的 class 的默认 minor.major version 吧。（又走到 Sun 的网站上翻腾出我从来都没用过的古董来）

![20151030110206.png](https://dn-biezhi.qbox.me/2015/10/3226308965.png)

当然你也可以用其他方法查看版本号，比如javap -verbose XXXX(class名)。

        那么现在如果碰到这种问题该知道如何解决了吧，还会像我所见到有些兄弟那样，去找个 1.4 的 JDK 下载安装，然后用其重新编译所有的代码吗？且不说这种方法的繁琐，而且web应用程序还不一定能成功，其实大可不必如此费神，我们一定还记得 javac 还有个 -target 参数，对啦，可以继续使用 1.5 JDK，编译时带上参数 -target 1.4 -source 1.4 就 OK 啦，不过你一定要对哪些 API 是 1.5 JDK 加入进来的了如指掌，不能你的 class 文件拿到 JVM 1.4 下就会 method not found。目标 JVM 是 1.3 的话，编译选项就用 -target 1.3 -source 1.3 了。

      相应的如果使用 ant ，它的 javac 任务也可对应的选择 target 和 source

<javac target="1.4" source="1.4" ............................/>

如果是在开发中，可以肯定的是现在真正算得上是 JAVA IDE 对于工程也都有编译选项设置目标代码的。例如 Eclipse 的项目属性中的 Java Compiler 设置，如图：

![2.jpg](https://dn-biezhi.qbox.me/2015/10/4103987148.jpg)

其实理解 major.minor 就像是我们可以这么想像，同样是微软件的程序，32 位的应用程序不能拿到 16 位系统中执行那样。

如果我们发布前了解到目标 JVM 版本，知道怎么从 java class 文件中看出 major.minor 版本来，就不用等到服务器报出异常才着手去解决，也就能预知到可能发生的问题。

其他时候遇到这个问题应具体解决，总之问题的根由是低版本的 JVM 无法加载高版本的 class 文件造成的，找到高版本的 class 文件处理一下就行了。
