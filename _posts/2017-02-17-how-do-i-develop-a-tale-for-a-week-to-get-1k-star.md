---
layout:     post
title:      "我如何在一周开发出Tale应用获得1k star"
catalog:    true
tags:
    - java
    - tale
    - blade
---

## 缘起

我是一个java开发者，几乎使用了所有主流的博客系统，包括静态博客和php系列，我在看java中有没有同样优秀的博客平台，我找到了 [jpress](https://github.com/JpressProjects/jpress) 和 [mblog](http://git.oschina.net/mtons/mblog)。

<!-- more -->

打开 jpress 的演示站点是这样的：

![jrepssdemo](https://ooo.0o0.ooo/2017/03/02/58b7c5dd7b5b6.png)

于是我分别打开了它官网的文档和视频教程，发现文档好像一直没有写，教程下面讲到《精通JFinal》，omg的。不过看起来功能确实还不错，但是我没有打算使用的心情了。

之前在某qq群有别人推荐mblog，我看了一下它的介绍是这样的：

> JDK8
数据库MySQL
SSH (Spring、SpringMVC、Hibernate）
安全权限 Shiro
搜索工具 Lucene
缓存 Ehcache
视图模板 Velocity
其它 Jsoup、fastjson、GraphicsMagick
jQuery、Seajs
Bootstrap 前端框架
UEditor/Markdown编辑器
font-wesome 字体/图标

可以看到用到了很多企业级开发的技术，搜索引擎、权限管理、竟然还有 `hibernate`。。

![](http://p3.pstatp.com/large/326000223684499a333)

显然这不是我想要的，部署起来的成本太高了。

---

## 定位和思考

于是我也自己开发一个博客系统，我认为每个坚持写博客的人都是有故事有分享精神的，项目起名简单易记住最好，于是找到 `tale` 这个单词，英文翻译是 `故事`，发音为 `[teɪl]`，用中文读 `塌了` 也蛮有趣的，就选它了。

我对个人博客的想法是这样的，目前来看它应该是单用户的，我见过写博客文章最多的也就 `1000` 篇左右，于是 mysql 的 `like` 完全可以做到查询搜索，不用搜索引擎（当然分词搜索另当别论，可以以插件方式出现）；它应该发布简单，体积小；它的设计简洁，易维护；它的UI美观，颜值高；它的速度很快，高性能。

---

## 技术选型

根据我上面所说的开发这样一个博客系统，如果还想在短期内完成它，选择 `SSM`, `SSH` 这种企业级框架肯定是有些臃肿了，不是说它做不了，你要用生成器什么的也可以，但它就是企业级架构，Spring3x 和 Spring4x 都够一些人折腾了。是 JFinal 吗？这看起来是个不错的选择，社区支持度也很高，但部署的方式不太优雅，还是要用tomcat的方式，我想像 `SpringBoot` 那样发布一个 `jar` 包运行一行 `java -jar tale-1.0.jar` 就可以跑起来的，我决定用自己在15年发起的 `blade` 作为web框架，具体的可以看 [这里](https://github.com/biezhi/blade) 总之它能解决我遇到的那些问题，又是我熟悉的，也因为是自己研发的，想让它走的更远些。

后台编辑器选择了github上修改自 `typecho` 的后台编辑器，也许并不是最酷的，我非专业前端，目前来看够用，我们可以以后再优化它，数据库选择了mysql，`tale` 是带后台的博客系统，当然我也考虑过用java的一款本地数据库 `mapdb`，这样开发者就不用创建数据库了，但数据备份是个问题，以后再深究这个。

- mvc框架：blade
- 数据库：mysql
- 后台模板：bootstrap
- 默认主题模板：pingshu
- 模板引擎：jetbrick template

其实技术也就是我们日常开发的那些技术，只不过看你如何用好它了。

---

## 开发细节

### markdown如何存储和解析的？

我将markdown的源码存储在数据库中，使用了 [commonmark](https://github.com/atlassian/commonmark-java) 这个库进行解析，在前台展示的时候将markdown解析为html即可。

### 你是如何支持emoji表情的？

有些开发者处理过应该知道 `mysql` 的 `utf8` 编码的一个字符最多3个字节，但是一个emoji表情为4个字节，所以utf8不支持存储emoji表情。但是utf8的超集utf8mb4一个字符最多能有4字节，所以能支持emoji表情的存储。但是我认为让使用者去修改这个配置太繁琐了，容易出错，我们直接用程序处理吧，于是乎找到一个库 [emoji-java](https://github.com/vdurmont/emoji-java)，它可以在emoji表情输入进来的时候被解析为形如 `:smile:` 这样的格式，我们在使用的时候再将它转换回去就好了。

### 更多细节

开发中往往有无数的细节和代码片段，其实都是日常用的，这里无法一一概述，如果你有疑问可以加入开发者讨论群 `1013565` 讨论，当然QQ群并非最好的交流方式，你乐意写邮件也可以联系我 `biezhi.me[AT]gmail.com`。

---

## 编写文档

一个好的项目不一定实力多么强，但是至少要有一个有力的 `README` 文档和简单易懂的快速入门说明，这会让你的项目在短时间内更受关注。我认为一个好的 `README` 文档可以是这样：

## 标题

> 一句话描述，简单易懂，说明用意

![fbi-warning.jpg](https://ooo.0o0.ooo/2017/03/02/58b7d6fc315f2.jpg)

*最好来一张高清无码有逼格的大图*

## 特性

- 史上最强瑞文
- 打不死的小强
- balabala

## 使用/预览图

*这里可以根据你的项目类型来安排*

## 开源协议

*标注你的开源协议*

[Apache2](https://github.com/biezhi/blade/blob/master/LICENSE)

## 联系方式

- 论坛：如果有的话
- 邮件：biezhi.me[AT]gmail.com
- QQ群：1013565

---

## 推广项目

虽然花了点时间做出了这个东西，但是不知道反响如何于是我在如下几个平台发布了这则消息：

- [v2ex](https://www.v2ex.com/t/343159)：创意工作者的社区
- [v2mm](https://v2mm.tech/topic/443/tale%E5%8D%9A%E5%AE%A2%E7%B3%BB%E7%BB%9F-%E8%AE%A9%E6%AF%8F%E4%B8%80%E4%B8%AA%E6%9C%89%E6%95%85%E4%BA%8B%E7%9A%84%E4%BA%BA%E6%9B%B4%E5%A5%BD%E7%9A%84%E8%A1%A8%E8%BE%BE%E6%83%B3%E6%B3%95/2)：看起来也是个不错的交流社区
- [掘金](https://gold.xitu.io/post/58b17cf31b69e60058a90a37)：一个只有高手分享的社区
- [开发者头条](https://toutiao.io/posts/4is9a1)：程序员分享平台

刚开始的时候应该是因为界面还算美观，也收到不少开发者的青睐，v2ex和掘金，开发者头条为它带来了不少的流量，在发布后的第二，第三天登上了github的 `Trending` 榜单，这时候收集到 `500` 左右的star；于是开始收到有人使用和反馈了，发现活跃度蛮高的，根据大家提出的意见出了简单的视频教程，修复了自己没发现的一些bug。不过在这一周里也是我打满鸡血的一周，今天项目在github收到了 `1k` star。
![star](https://ooo.0o0.ooo/2017/03/02/58b7d60ab259d.png)
也让我对后面的开发更有动力，如果你觉得这个项目还不错也可以 [捐赠] 我，该博客系统面向所有用户，开源免费。 也有些开发者想听我谈谈如何开发这个系统的，于是写下了本文，还望各位看官轻喷。

---
## 反馈

tale在短时间内已经有很多的反馈，我们使用github的issues进行记录

![ar3.png](https://ooo.0o0.ooo/2017/03/02/58b7df503f4a3.png)

联合开发者一起贡献，有不少开发者向 `tale` 发起了 `PR`

![ar4.png](https://ooo.0o0.ooo/2017/03/02/58b7df4f86b76.png)

---

## 感谢

非常感谢最近帮助我的小伙伴，感谢大家的反馈和 `Pull Request` 没有你们也就没有更好更优秀的博客平台。

- [pkwenda](https://github.com/pkwenda)
- [5hmybmny](https://github.com/hmybmny)
- [trytrytryiii](https://github.com/trytrytryiii)
- [Yifei0727](https://github.com/Yifei0727)

