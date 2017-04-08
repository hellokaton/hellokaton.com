---
layout: post
title: 查看mysql连接数
categories: mysql
tags: mysql
---

### 1、查看当前所有连接的详细资料:

```bash
mysqladmin -uroot -proot processlist
```

### 2、只查看当前连接数(Threads就是连接数.):

```bash
mysqladmin -uroot -proot status
```

### 3、修改mysql最大连接数：

> 打开my.ini，修改max_connections=100(默认为100)