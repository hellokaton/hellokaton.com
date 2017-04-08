---
layout: post
title: MySQL新建用户,授权,删除用户,修改密码
categories: mysql
tags: mysql
---

首先要声明一下：一般情况下，修改MySQL密码，授权，是需要有mysql里的root权限的。

注：本操作是在WIN命令提示符下，phpMyAdmin同样适用。
    用户：phplamp  用户数据库：phplampDB

<!-- more -->

1.新建用户。

```bash
//登录MYSQL
@>mysql -u root -p
@>密码
//创建用户
mysql> insert into mysql.user(Host,User,Password) values("localhost","phplamp",password("1234"));
//刷新系统权限表
mysql>flush privileges;
```

这样就创建了一个名为：phplamp  密码为：1234  的用户。

然后登录一下。

```bash
mysql>exit;
@>mysql -u phplamp -p
@>输入密码
mysql>登录成功
```

2.为用户授权。

//登录MYSQL（有ROOT权限）。我里我以ROOT身份登录.

```bash
@>mysql -u root -p
@>密码
```

//首先为用户创建一个数据库(phplampDB)

`mysql>create database phplampDB;`

//授权phplamp用户拥有phplamp数据库的所有权限。

```bash
>grant all privileges on phplampDB.* to phplamp@localhost identified by '1234';
```

//刷新系统权限表

```bash
mysql>flush privileges;
mysql>其它操作
```

如果想指定部分权限给一用户，可以这样来写:

```bash
mysql>grant select,update on phplampDB.* to phplamp@localhost identified by '1234';
//刷新系统权限表。
mysql>flush privileges;
```

3.删除用户。

```bash
@>mysql -u root -p
@>密码
mysql>DELETE FROM user WHERE User="phplamp" and Host="localhost";
mysql>flush privileges;

//删除用户的数据库
mysql>drop database phplampDB;
```

4.修改指定用户密码。

```bash
@>mysql -u root -p
@>密码
mysql>update mysql.user set password=password('新密码') where User="phplamp" and Host="localhost";
mysql>flush privileges;
```