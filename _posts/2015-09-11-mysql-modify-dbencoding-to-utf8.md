---
layout: post
title: MySql修改数据库编码为UTF8
categories: mysql
tags: mysql 编码
---

mysql 创建 数据库时指定编码很重要，很多开发者都使用了默认编码，乱码问题可是防不胜防。制定数据库的编码可以很大程度上避免倒入导出带来的乱码问题。

网页数据一般采用UTF8编码，而数据库默认为latin 。我们可以通过修改数据库默认编码方式为UTF8来减少数据库创建时的设置，也能最大限度的避免因粗心造成的乱码问题。
   
我们遵循的标准是，数据库，表，字段和页面或文本的编码要统一起来

<!-- more -->

我们可以通过命令查看数据库当前编码：  

```bash
mysql> SHOW VARIABLES LIKE 'character%';
```

发现很多对应的都是 latin1，我们的目标就是在下次使用此命令时latin1能被UTF8取代。

**第一阶段：**

mysql设置编码命令
    
```bash
SET character_set_client = utf8;
SET character_set_connection = utf8;
SET character_set_database = utf8;
SET character_set_results = utf8;
SET character_set_server = utf8;
```

然后 `mysql> SHOW VARIABLES LIKE 'character%';` 你可以看到全变为 utf8
但是，这只是一种假象
此种方式只在当前状态下有效，当重启数据库服务后失效。
所以如果想要不出现乱码只有修改my.ini文件，
从`my.ini`下手（标签下没有的添加，有的修改）

```bash
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
default-character-set=utf8
```

以上3个section都要加default-character-set=utf8，平时我们可能只加了mysqld一项。
然后重启mysql，执行

```bash
mysql> SHOW VARIABLES LIKE 'character%';
```

确保所有的Value项都是utf8即可