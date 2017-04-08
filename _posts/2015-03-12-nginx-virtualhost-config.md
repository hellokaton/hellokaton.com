---
layout: post
title: Nginx 虚拟主机 VirtualHost 配置
categories: nginx
tags: nginx
---

## 增加 Nginx 虚拟主机

这里假设大家的 Nginx 服务器已经安装好, 不懂的请阅读各 Linux 发行版的官方文档或者 LNMP 的安装说明. 配置 Virtual host 步骤如下:

<!-- more -->

##### 1. 进入 /usr/local/nginx/conf/vhost 目录, 创建虚拟主机配置文件 demo.neoease.com.conf ({域名}.conf).

##### 2. 打开配置文件, 添加服务如下:

```bash
server {
    listen       80;
    server_name demo.neoease.com;
    index index.html index.htm index.php;
    root  /var/www/demo_neoease_com;
 
    log_format demo.neoease.com '$remote_addr - $remote_user [$time_local] $request'
    '$status $body_bytes_sent $http_referer '
    '$http_user_agent $http_x_forwarded_for';
    access_log  /var/log/demo.neoease.com.log demo.neoease.com;
}
```
<!--more-->

##### 3. 打开 Nginx 配置文件 `/usr/local/nginx/conf/nginx.conf`, 在 http 范围引入虚拟主机配置文件如下:

````bash
include vhost/*.conf;
```

##### 4. 重启 `Nginx` 服务, 执行以下语句.

```bash
service nginx restart
```

## 让 `Nginx` 虚拟主机支持 PHP

在前面第 2 步的虚拟主机服务对应的目录加入对 PHP 的支持, 这里使用的是 FastCGI, 修改如下.

```bash
server {
    listen       80;
    server_name demo.neoease.com;
    index index.html index.htm index.php;
    root  /var/www/demo_neoease_com;
 
    location ~ .*\.(php|php5)?$ {
        fastcgi_pass unix:/tmp/php-cgi.sock;
        fastcgi_index index.php;
        include fcgi.conf;
    }
 
    log_format demo.neoease.com '$remote_addr - $remote_user [$time_local] $request'
    '$status $body_bytes_sent $http_referer '
    '$http_user_agent $http_x_forwarded_for';
    access_log  /var/log/demo.neoease.com.log demo.neoease.com;
}
```

## 图片防盗链

图片作为重要的耗流量大的静态资源, 可能网站主并不希望其他网站直接引用, Nginx 可以通过 referer 来防止外站盗链图片.

```bash
server {
    listen       80;
    server_name demo.neoease.com;
    index index.html index.htm index.php;
    root  /var/www/demo_neoease_com;
 
    # 这里为图片添加为期 1 年的过期时间, 并且禁止 Google, 百度和本站之外的网站引用图片
    location ~ .*\.(ico|jpg|jpeg|png|gif)$ {
        expires 1y;
        valid_referers none blocked demo.neoease.com *.google.com *.baidu.com;
        if ($invalid_referer) {
            return 404;
        }
    }
 
    log_format demo.neoease.com '$remote_addr - $remote_user [$time_local] $request'
    '$status $body_bytes_sent $http_referer '
    '$http_user_agent $http_x_forwarded_for';
    access_log  /var/log/demo.neoease.com.log demo.neoease.com;
}
```

## WordPress 伪静态配置

如果将 WordPress 的链接结构设定为 `/%postname%/`, `/%postname%.html` 等格式时, 
需要`rewrite URL`, WordPress 提供 Apache 的 .htaccess 修改建议, 但没告知 Nginx 该如何修改.
我们可以将 WordPress 的虚拟主机配置修改如下:

```bash
server {
    listen       80;
    server_name demo.neoease.com;
    index index.html index.htm index.php;
    root  /var/www/demo_neoease_com;
 
    location / {
        if (-f $request_filename/index.html){
            rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
    }
    rewrite /wp-admin$ $scheme://$host$uri/ permanent;
 
    location ~ .*\.(php|php5)?$ {
        fastcgi_pass unix:/tmp/php-cgi.sock;
        fastcgi_index index.php;
        include fcgi.conf;
    }
 
    log_format demo.neoease.com '$remote_addr - $remote_user [$time_local] $request'
    '$status $body_bytes_sent $http_referer '
    '$http_user_agent $http_x_forwarded_for';
    access_log  /var/log/demo.neoease.com.log demo.neoease.com;
}
```

LNMP 套件在提供了 WordPress 为静态配置文件 `/usr/local/nginx/conf/wordpress.conf`, 在虚拟主机配置的 server 范围引用如下即可.

```bash
include wordpress.conf;
```

如果你使用 LNMP 套件, 进入 WordPress 后台发现会出现 404 页面, wp-admin 后面缺少了斜杆 /, 
请在 wordpress.conf 最后添加以下语句:

```bash
rewrite /wp-admin$ $scheme://$host$uri/ permanent;
```

## 后话

一直以来, 我主要在用 Apache, 自从去年从 MT 搬家到 Linode VPS 之后, 发现服务器压力很大, 每隔几天就要宕机一次, 在胡戈戈的协助下转成了 Nginx, 大半年了一直很稳定.

相对 Apache, Nignx 有更加强大的并发能力, 而因为他对进程管理耗用资源也比较少. 而 Apache 比 Nginx 有更多更成熟的可用模块, bug 也比较少. 卖主机的 IDC 选择 Nignx, 因为高并发允许他们创建更多虚拟主机空间更来钱; 淘宝也因此改造 Nignx (Tengine) 作为 CDN 服务器, 可承受更大压力.
