---
title: 一行代码引发的血案
date: 2017-08-11
tags: ["java"]
---

最近我写的程序出了一个 `StackOverflowError` 的错误，写出来分析一下原因并提出解决方法。
想想还蛮激动的，人生中第一次写 `StackOverflowError` 这种错误。

<!-- more -->

{% image /static/img/article/oh-my-stackoverflowerror.png 380 314 对方不想和你说话并向你抛了个异常 %}

## StackOverflowError是什么？

```java
/**
 * 哦 我的栈溢出错误
 *
 * @author biezhi
 * @date 2017/8/11
 */
public class OhMyStackOverflowError {

    public static void sixsixsix(){
        sixsixsix();
    }

    public static void main(String[] args) {
        sixsixsix();
    }

}
```

## OutOfMemoryError如何触发的?

## 如何应对和解决这些问题
