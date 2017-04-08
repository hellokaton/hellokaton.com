---
layout: post
title: JavaWeb之Filter工作原理
categories: java
tags: filter servlet
---

## 概述

Filter是Javaweb中的过滤器，Web服务器根据应用程序配置文件设置的过滤规则进行检查，若客户请求满足过滤规则，则对客户请求／响应进行拦截，对请求头和请求数据进行检查或改动，并依次通过过滤器链，最后把请求／响应交给请求的Web资源处理。请求信息在过滤器链中可以被修改，也可以根据条件让请求不发往资源处理器，并直接向客户机发回一个响应。当资源处理器完成了对资源的处理后，响应信息将逐级逆向返回。同样在这个过程中，用户可以修改响应信息，从而完成一定的任务。

<!-- more -->

## 生命周期

1. 在应用启动的时候就进行装载Filter类而servlet是在请求时才创建(但filter与Servlet的load-on-startup配置效果相同)。

2. 器创建好Filter对象实例后，调用init方法。接着被Web容器保存进应用级的集合容器中去了等待着，用户访问资源。

3. 当用户访问的资源正好被Filter的url-pattern拦截时，容器会取出Filter类调用doFilter方法，下次或多次访问被拦截的资源时，Web容器会直接取出指定Filter对象实例调用doFilter方法(Filter对象常驻留Web容器了)。

4. 当应用服务被停止或重新装载了，则会执行Filter的destroy方法，Filter对象销毁。

## 配置

Filter也是在web.xml中一个常用的配置项，可以通过如下进行配置

```xml
<filter>  
    <filter-name>sensitiveFilter</filter-name>  
    <filter-class>com.biezhi.SensitiveFilter</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>sensitiveFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

```java
package com.biezhi.filter;

import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/*
 * 敏感词汇过滤
 */
public class SensitiveFilter implements Filter {
　　
　　@Override
　　public void doFilter（ServletRequest request, ServletResponse response, FilterChain chain） 
    throws IOException, ServletException {
　　    // 转换成实例的请求和响应对象
　　    HttpServletRequest req = （HttpServletRequest） request;
　　    HttpServletResponse resp = （HttpServletResponse） response;
　　    // 获取评论并屏蔽关键字
　　    String str = req.getParameter（"str"）；
　　    str = str.replace（"被就业", "***"）；
　　    // 重新设置参数
　　    req.setAttribute（"str", str）；
　　    // 继续执行
　　    chain.doFilter（request, response）；
　　}

　　@Override
　　public void init（FilterConfig filterConfig） throws ServletException{}
    @Override
　　public void destroy(){}
｝
```

实际上Filter可以完成Servlet同样的工作，甚至比Servlet更加灵活，因为它除了提供了request和response对象
外，还提供了一个FilterChain对象，这个对象可以让我们更加灵活的控制请求的流转。下面是类图：

![过滤器类图.png](https://dn-biezhi.qbox.me/2015/09/777796409.png)

## 执行流程

下面是Filter的执行流程图：

![执行流程图.png](https://dn-biezhi.qbox.me/2015/09/3031115299.png)

在Tomcat容器中，FilterConfig和FilterChain的实现类分别是ApplocationFilterConfig和ApplicationFilterChain，Filter是一个接口，由用户自己实现，只要实现Filter接口的三个方法即可，这三个方法和Servlet中的类似。只不过还有一个ApplocationFilterChain类，这个类可以将多个Filter串联起来，组成一个链，这个链与jetty中的Handler链有异曲同工之妙，下面是Filter接口中的三个方法：

- `init(FilterConfig)`
初始化方法，在用户自定义的Filter初始化时被调用，它与Servlet的init方法作用相同，FilterConfig与ServletConfig也类似，除了都能获取到容器的上下文(Context)之外，还可以获取Filter在初始化时设置的<init-param>参数值。

- `doFilter(SerlvetRequest, ServletResponse, FilterChain)`
每个请求进来的时候这个方法都会被调用，并在Servlet的service方法执行之前。而FilterChain就代表当前的整个请求链，所以通过调用FilterChain的doFilter方法可以继续将请求传递下去。如果想拦截这个请求，可以不调用FilterChian的doFilter，那么这个请求就返回了，所以Filter是一种责任链模式。

- `destory`
当Filter被销毁的时候调用该方法。注意，当web容器调用这个方法的时候，会再调用一次doFilter方法。

Filter类的核心还是doFilter中传递的FilterChain对象，这个对象保存了到最终的Servlet对象的所有Filter对象，这些对象都保存在ApplicationFilterChain对象的filter数组中。在FilterChain链上每执行一个Filter对象，数组的当前计数都会加1，直到计数等于数组的长度，当FilterChain上的所有Filter对象都执行完之后，就会执行最终的Servlet。所以在ApplicationFilterChain对象中会持有Servlet对象的引用。下图是Filter对象执行的时序图：
![filter时序图.png](https://dn-biezhi.qbox.me/2015/09/3226074832.png)

Filter存在的意义就像你要找一个对象，对象是你的最终目标，但是会提供一个过滤机制让你进行筛选合适的妹子~在中途可以做一些拦截操作，比如太丑拒绝。。总之它可以在你的途中增加或者减少一些操作。
