---
layout: post
title: 为Nginx目录设置访问密码
categories: nginx
tags: nginx
---

可以使用以下这个python脚本生成：

```
http://trac.edgewall.org/export/10770/trunk/contrib/htpasswd.py
```

<!-- more -->

执行命令：

```bash
chmod 777 htpasswd.py
./htpasswd.py -c -b htpasswd username password
```

其中htpasswd是生成的文件名

**修改nginx的conf**

修改nginx.conf或者所要设置的vhost的conf，加入如下语句：

```bash
location  ^~ / {
	auth_basic "Password";
	auth_basic_user_file /usr/local/nginx/conf/htpasswd;
}
```