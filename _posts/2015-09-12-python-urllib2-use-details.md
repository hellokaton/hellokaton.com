---
layout: post
title: Python 标准库 urllib2 的使用细节
categories: python
tags: python urllib2
---

Python 标准库中有很多实用的工具类，但是在具体使用时，标准库文档上对使用细节描述的并不清楚，比如 urllib2 这个 HTTP 客户端库。这里总结了一些 urllib2 的使用细节。

- Proxy 的设置
- Timeout 设置
- 在 HTTP Request 中加入特定的 Header
- Redirect
- Cookie
- 使用 HTTP 的 PUT 和 DELETE 方法
- 得到 HTTP 的返回码
- Debug Log

<!-- more -->

## Proxy 的设置

urllib2 默认会使用环境变量 http_proxy 来设置 HTTP Proxy。如果想在程序中明确控制 Proxy 而不受环境变量的影响，可以使用下面的方式

```python
import urllib2

enable_proxy = True
proxy_handler = urllib2.ProxyHandler({"http" : 'http://some-proxy.com:8080'})
null_proxy_handler = urllib2.ProxyHandler({})

if enable_proxy:
    opener = urllib2.build_opener(proxy_handler)
else:
    opener = urllib2.build_opener(null_proxy_handler)

urllib2.install_opener(opener)
```

## Timeout 设置

在老版 Python 中，urllib2 的 API 并没有暴露 Timeout 的设置，要设置 Timeout 值，只能更改 Socket 的全局 Timeout 值。

```python
import urllib2
import socket

socket.setdefaulttimeout(10) # 10 秒钟后超时
urllib2.socket.setdefaulttimeout(10) # 另一种方式
```
在 Python 2.6 以后，超时可以通过 urllib2.urlopen() 的 timeout 参数直接设置。
```python
import urllib2
response = urllib2.urlopen('http://www.google.com', timeout=10)
```

## 在 HTTP Request 中加入特定的 Header

要加入 header，需要使用 Request 对象：

```python
import urllib2

request = urllib2.Request(uri)
request.add_header('User-Agent', 'fake-client')
response = urllib2.urlopen(request)
```

对有些 header 要特别留意，服务器会针对这些 header 做检查

- User-Agent : 有些服务器或 Proxy 会通过该值来判断是否是浏览器发出的请求

- Content-Type : 在使用 REST 接口时，服务器会检查该值，用来确定 HTTP Body 中的内容该怎样解析。常见的取值有：

- application/xml ： 在 XML RPC，如 RESTful/SOAP 调用时使用
- application/json ： 在 JSON RPC 调用时使用
- application/x-www-form-urlencoded ： 浏览器提交 Web 表单时使用

在使用服务器提供的 RESTful 或 SOAP 服务时， Content-Type 设置错误会导致服务器拒绝服务

## Redirect

urllib2 默认情况下会针对 HTTP 3XX 返回码自动进行 redirect 动作，无需人工配置。要检测是否发生了 redirect 动作，只要检查一下 Response 的 URL 和 Request 的 URL 是否一致就可以了。

```python
import urllib2
response = urllib2.urlopen('http://www.google.cn')
redirected = response.geturl() == 'http://www.google.cn'
```

如果不想自动 redirect，除了使用更低层次的 httplib 库之外，还可以自定义 HTTPRedirectHandler 类。

```python
import urllib2

class RedirectHandler(urllib2.HTTPRedirectHandler):
    def http_error_301(self, req, fp, code, msg, headers):
        pass
    def http_error_302(self, req, fp, code, msg, headers):
        pass

opener = urllib2.build_opener(RedirectHandler)
opener.open('http://www.google.cn')
```

## Cookie

urllib2 对 Cookie 的处理也是自动的。如果需要得到某个 Cookie 项的值，可以这么做：

```python
import urllib2
import cookielib

cookie = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
response = opener.open('http://www.google.com')
for item in cookie:
    if item.name == 'some_cookie_item_name':
        print item.value
```

## 使用 HTTP 的 PUT 和 DELETE 方法

urllib2 只支持 HTTP 的 GET 和 POST 方法，如果要使用 HTTP PUT 和 DELETE ，只能使用比较低层的 httplib 库。虽然如此，我们还是能通过下面的方式，使 urllib2 能够发出 PUT 或 DELETE 的请求：

```python
import urllib2

request = urllib2.Request(uri, data=data)
request.get_method = lambda: 'PUT' # or 'DELETE'
response = urllib2.urlopen(request)
```

这种做法虽然属于 Hack 的方式，但实际使用起来也没什么问题。

## 得到 HTTP 的返回码

对于 200 OK 来说，只要使用 urlopen 返回的 response 对象的 getcode() 方法就可以得到 HTTP 的返回码。但对其它返回码来说，urlopen 会抛出异常。这时候，就要检查异常对象的 code 属性了：

```python
import urllib2
try:
    response = urllib2.urlopen('http://restrict.web.com')
except urllib2.HTTPError, e:
    print e.code
```

## Debug Log

使用 urllib2 时，可以通过下面的方法把 debug Log 打开，这样收发包的内容就会在屏幕上打印出来，方便调试，有时可以省去抓包的工作

```python
import urllib2

httpHandler = urllib2.HTTPHandler(debuglevel=1)
httpsHandler = urllib2.HTTPSHandler(debuglevel=1)
opener = urllib2.build_opener(httpHandler, httpsHandler)

urllib2.install_opener(opener)
response = urllib2.urlopen('http://www.google.com')
```