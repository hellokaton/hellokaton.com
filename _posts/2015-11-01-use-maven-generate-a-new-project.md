---
layout: post
title: 使用Maven的archetype快速生成一个新项目
categories: java
tags: maven
---

Maven的archetype Plugin可能大家都听过，但不一定都能很好地用好它。缺省地如果你使用

```sh
mvn archetype:generate
```

会从maven的Repository里查找所有支持的arche types,大概有500~600个。正因为是太多了，所以查找起来很是不方便。

<!-- more -->

其实平时常用的arche type也就那么几个。像我会用到的：

1. simple start 
2. web app
3. Groovy basic

很自然的就会考虑，是不是能什么简便的方法只需要从这3个组成的list里选择就可以了。 答案当然是： Yes

实现步骤如下：（本机的Maven Repository目录在D:\maven\repo ）

1. 使用 `mvn archetype:crawl` 命令，它会在 `D:\maven\repo` 目录下生成一个 `archetype-catalog.xml` 文件

2.将 `archetype-catalog.xml` 移到上一层目录，也就是 `D:\maven\repo`

3.这时再运行 `mvn archetype:generate -DarchetypeCatalog=local` 就可以达到你想要的目的了。

![maven_archetype.png][1]

是不是很方便啊。

Links：[http://maven.40175.n5.nabble.com/archetype-catalog-xml-location-archetype-crawl-versus-archetype-generate-td113741.html][2] 

想得到更全的 `archtetype-catalog.xml` 可以访问： [http://repo1.maven.org/maven2/archetype-catalog.xml][3]


  [1]: https://dn-biezhi.qbox.me/2015/11/4232924896.png
  [2]: http://maven.40175.n5.nabble.com/archetype-catalog-xml-location-archetype-crawl-versus-archetype-generate-td113741.html
  [3]: http://repo1.maven.org/maven2/archetype-catalog.xml