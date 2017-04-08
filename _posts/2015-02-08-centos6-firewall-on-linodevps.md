---
layout: post
title: Linode VPS上的Centos 6.4系统的防火墙
categories: linux
tags: linode vps centos
---

> Linode VPS Centos6 iptables错误Setting chains to policy ACCEPT: security raw nat[FAILED]filter的解决方法

<!-- more -->

近日在配置Linode VPS上的Centos 6.4系统的防火墙的时候，遇到以下错误：

```bash
service iptables restart
Setting chains to policy ACCEPT: security raw nat[FAILED]filter
```

经过搜索，明白是Linode官方在iptables里加了一个security的规则链，但Centos不支持。
而找到的解决方法是，编辑 `/etc/init.d/iptables`，找到：

```bash
for i in $tables; do
        echo -n "$i "
        case "$i" in
            raw)
                $IPTABLES -t raw -P PREROUTING $policy \
                    && $IPTABLES -t raw -P OUTPUT $policy \
                    || let ret+=1
```

<!--more-->

加入以下内容到*“case "$i" in”*下面：

```bash
security)
    $IPTABLES -t filter -P INPUT $policy \
        && $IPTABLES -t filter -P OUTPUT $policy \
        && $IPTABLES -t filter -P FORWARD $policy \
        || let ret+=1
    ;;
```

最终版本类似：

```bash
for i in $tables; do
    echo -n "$i "
    case "$i" in
        security)
            $IPTABLES -t filter -P INPUT $policy \
                && $IPTABLES -t filter -P OUTPUT $policy \
                && $IPTABLES -t filter -P FORWARD $policy \
                || let ret+=1
            ;;
        raw)
            $IPTABLES -t raw -P PREROUTING $policy \
                && $IPTABLES -t raw -P OUTPUT $policy \
                || let ret+=1
            ;;
```

保存后，重启则可

```bash
$ service iptables restart
```