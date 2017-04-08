---
layout: post
title: 使用wget下载xcode
categories: [linux, ios]
tags: xcode
---

由于家里网速比较悲伤。。。只能先用服务器下载好，然后传到七牛的空间里这样比较快。我喜欢自己下载官方的，so 没有直接去别的CDN或者站点去下载。

<!-- more -->

在浏览器中直接下载xocde是很简单的，只要登录你的appleid，然后找到 [https://developer.apple.com/xcode/download/](https://developer.apple.com/xcode/download/) 即可下载，我一看卧槽4个G，家里肯定吃不消了。。

于是用自己日本的服务器来下载，直接使用 `wget` 下载 [http://adcdownload.apple.com/Developer_Tools/Xcode_7.3_beta_5/Xcode_7.3_beta_5.dmg](http://adcdownload.apple.com/Developer_Tools/Xcode_7.3_beta_5/Xcode_7.3_beta_5.dmg) 是行不通的，有cookie验证，我们可以使用chrome扩展把cookie下载下来然后使用wget，这招也可以用到其他的下载中，比如JDK。

<!--more-->

### 步骤

1. 安装 [cookies.txt](https://chrome.google.com/webstore/detail/cookiestxt/njabckikapfpffapmjgojcnbfjonfjfg?hl=cn) Chrome 扩展(需FQ)
2. 登录到[Apple Developer](https://developer.apple.com)并点击下载Xcode
3. 运行cookie.txt扩展并下载cookie文件到本地
4. 将cookie文件传到服务器然后运行如下命令
```sh
wget --load-cookies=cookies.txt -c http://adcdownload.apple.com/Developer_Tools/Xcode_7.3_beta_5/Xcode_7.3_beta_5.dmg
```

上传到七牛的空间可以看[这里](http://developer.qiniu.com/code/v6/tool/qrsctl.html)的文档，这里就不吐口水了。