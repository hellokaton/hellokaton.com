---
layout: post
title: 用node-webkit把web应用打包成桌面应用
categories: nodejs
tags: node-webkit
---

[node-webkit](https://github.com/rogerwang/node-webkit)是一个Chromium和node.js上的结合体，通过它我们可以把建立在chrome浏览器和node.js上的web应用打包成桌面应用，而且还可以跨平台的哦。很显然比起传统的桌面应用，在某些特定领域用html5+css3+js开发的web应用更加简单和高效，而且还可以使用node.js的功能，所以node-webkit还是很有用处的。

<!-- more -->

下面我通过一个简单的demo来介绍怎么样把一个web应用打包成一个可执行文件（这里只介绍windows环境）

首先新建一个index.html文件，作为我们这个demo的入口页面，我们暂且就把这个页面当成一个完整的web应用吧。内容随便写点什么，比如：

![101541125232422.png](https://dn-biezhi.qbox.me/2015/09/3906849872.png)

然后创建配置文件 package.json,内容如下：

![101541131789320.png](https://dn-biezhi.qbox.me/2015/09/965193666.png)

其中的main属性就是用来指定入口文件的，这个属性的值可以是本地文件，也可以是远程网址，这样就相当于可以把一个远程的web应用直接变为一个桌面应用了。

除了name与main这两个属性外，还有很多其他有用的属性可以配置，比如指定应用的图标，显不显示浏览器的工具栏，指定浏览器的初始大小等等，具体的配置参数文档可看这里[https://github.com/rogerwang/node-webkit/wiki/Manifest-format](https://github.com/rogerwang/node-webkit/wiki/Manifest-format)

现在我们有了两个文件了。

![101541140418221.png](https://dn-biezhi.qbox.me/2015/09/3848583289.png)

然后将`index.html`和`package.json`这两个文件压缩到一个zip压缩包里，命名为`app.zip`

![101541178215610.png](https://dn-biezhi.qbox.me/2015/09/1443052314.png)

现在app.zip这个压缩包里的内容应该是这样的：

![101541197742941.png](https://dn-biezhi.qbox.me/2015/09/586189206.png)

然后把app.zip这个文件的扩展名改为nw,变为 app.nw

![101541204305541.png](https://dn-biezhi.qbox.me/2015/09/2991353672.png)

然后下载一个windows版本的node-webkit,解压后得到一个文件夹：

![101541212229712.png](https://dn-biezhi.qbox.me/2015/09/2856560818.png)

之后我们之前得到的app.nw这个文件就可以用nw.exe来执行了，直接把app.nw拖到nw.exe上就可以了。运行结果如下：

![101541220397828.png](https://dn-biezhi.qbox.me/2015/09/3554349071.png)

跟在chrome中打开index.html这个页面的效果差不多，当然你可以通过配置package.json这个文件，来隐藏浏览器的工具栏或边框，来使它更像是一个桌面软件。

 

因为nw文件的运行需要node-webkit环境的支持，所以我们还需要把app.nw这个文件跟node-webkit的环境文件一起打包成一个可执行文件。

首先打开windows的cmd,然后输入如下命令：
```bash
copy /b nw.exe+app.nw app.exe
```
注意文件路径要根据你的实际情况进行变动，这里假设`app.nw`放在了`node-webkit`的主文件夹里，然后输出的``也会在这个文件夹里。

执行命令后我们得到了`app.exe`这个可执行文件。

![101541229006728.png](https://dn-biezhi.qbox.me/2015/09/1191960116.png)

到了这步，我们已经得到了app.exe这个文件，但如果只有app.exe这个文件还是不够的，这个可执行文件的运行还需要几个dll文件的支持。

其中 nw.pak 与 icudt.dll 这个两个文件是必须要的。

ffmpegsumo.dll 文件是媒体支持文件，如果你的html页面中用到了<video>或<audio>或其它与媒体相关的东西，则必须带上这个文件。

libEGL.dll 和 libGLESv2.dll 这个两个文件则是使用webGL或GPU必须要的

最后我们得到的就是这样一个文件夹：

![101541238476414.png](https://dn-biezhi.qbox.me/2015/09/3259527669.png)

执行app.exe就可以运行我们的demo了。


但我们大多数人想的是给用户一个exe文件，用户就可以使用了，不用再附带一些其他文件。

嗯，所以我们还可以把app.exe跟其他的文件再打包一次，把上图中的所有文件变成一个可执行文件，用户只要得到这个文件，就能运行我们的应用了。

做这步我们需要一个软件叫[Enigma Virtual Box](http://enigmaprotector.com/en/aboutvb.html)，首先[下载](http://enigmaprotector.com/assets/files/enigmavb.exe)和安装这个软件，然后打开它。

然后在Enter Input File Name那里输入我们的app.exe的路径，在Enter Output File Name那里填写我们要把打包出来的可执行文件输出到哪里。最后是把除app.exe外的其它文件拖入到Files那里，遇到提示的话默认就可以了。

![101541248952302.png](https://dn-biezhi.qbox.me/2015/09/3277007985.png)

最后点击右下角的Process按钮，就大功告成了。

![101541260835531.png](https://dn-biezhi.qbox.me/2015/09/1326722595.png)

最后我们得到了一个 `app_boxed.exe` 的文件，只要把这个文件交给用户，用户就可以运行了。

`node-webkit`虽然方便，但有个很大的缺点是得到的可执行文件有点大，大家在可以在衡量利弊后决定使不使用。
