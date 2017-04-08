---
layout: post
title: nginx泛域名解析，实现多个二级域名
categories: nginx
tags: nginx 域名
---

利用nginx泛域名解析配置二级域名和多域名，实现二级域名子站，用户个性独立子域名。

主要针对用户独立子域名这种情况，不可能在配置里面将用户子域名写完，因此需要通过nginx泛解析方式。

配置方法：

```sh
server_name  ~^(?<subdomain>.+)\.yourdomain\.com$;
```

通过匹配subdomain即可。而在下面的可以通过$subdomain这个变量获取当前子域名称。

<!-- more -->

### 情况一：绑定子域名到统一目录，作为用户个性域名

这种情况下，只需要直接匹配就可以了，目录都是指向同一个地方的（一般）。

**配置实例：**

```sh
server {

    listen   80;
    server_name yourdomain.com www.yourdomain.cpm ~^(?<subdomain>.+)\.m\.yourdomain\.com$;

    index index.php index.html index.htm;
    set $root_path '/var/www/yanue.net';
    root $root_path;

    try_files $uri $uri/ @rewrite;

    location @rewrite {
        rewrite ^/(.*)$ /index.php?_url=/$1;
    }

    location ~ \.php {
            fastcgi_pass   127.0.0.1:9000;
    }

    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root $root_path;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

这样可以实现：

`user.m.yourdomain.com` 跳转到用户自己页面

当然跳转逻辑需要自己在程序里面去实现。

### 情况二：绑定子域名到不同目录（子站）

网站的目录结构为

```sh
html
├── bbs
└── www
```

html为nginx的安装目录下默认的存放源代码的路径。

bbs为论坛程序源代码路径

www为主页程序源代码路径

把相应程序放入上面的路径通过

http://www.youdomain.com 访问的就是主页

http://bbs.yourdomain.com 访问的就是论坛

其它二级域名类推。

**配置实例：**

```sh
server {
        listen       80;
        server_name  ~^(?<subdomain>.+)\.yourdomain\.com$;
        root   html/$subdomain; 
        index  index.html index.htm index.php;
        fastcgi_intercept_errors on;
        error_page  404      = /404.html;
        location / {
                # This is cool because no php is touched for static content.
                # include the "?$args" part so non-default permalinks doesn't
                # break when using query string
                try_files $uri $uri/ =404;
       }

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  domain $subdomain;
            include        fastcgi_params;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        location ~ /\.ht {
            deny  all;
        }
}
```