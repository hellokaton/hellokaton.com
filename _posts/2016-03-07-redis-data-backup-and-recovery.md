---
layout: post
title: Redis 数据备份与恢复
categories: nosql
tags: nosql redis
---

Redis `SAVE` 命令用于创建当前数据库的备份。

## 语法

redis Save 命令基本语法如下：

```sh
redis 127.0.0.1:6379> SAVE 
```

## 实例

```sh
redis 127.0.0.1:6379> SAVE 
OK
```

该命令将在 redis 安装目录中创建dump.rdb文件。

<!-- more -->

## 恢复数据

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：

```sh
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
```
以上命令 `CONFIG GET dir` 输出的 redis 安装目录为 `/usr/local/redis/bin`。

## Bgsave

创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。

### 实例

```sh
127.0.0.1:6379> BGSAVE

Background saving started
```