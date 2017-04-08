---
layout: post
title: REST API 的最佳入门教程
categories: 架构
tags: rest
---

如果你看到这里，你以前可能听说过API 和REST,然后你就会想:“这些都是什么东西？”。也许你已经了解过一些这方面的知识，但却不知道从何入手。在这个教程中，我将会诠释REST的基础以及如何给应用创建一个API（包括认证授权）

## 什么是API?

API是Application Programming Interface（应用程序接口）的缩写，它是拿来描述一个类库的特征或是如何去运用它。你个人收藏的类库也许包含有可用功能的“API文档”，那些必需的参数我们该怎么称呼它们？诸如此类等等。

然而，如今很多人参考API文档时，他们常常参考一种可能会通过网络分享你的应用数据HTTP API，例如，Twitter提供一个API能让用户在特定的格式下请求推文，以便用户方便导入到自己的应用程序中。这就是HTTP API的真正强大之处。它能够从多个应用程序中混搭数据到混合应用程序中，或是创建一个能增强使用他人应用体验的应用程序。

<!-- more -->

这样说吧，比如说我们有一个可以允许我们查看（view），创建（create），编辑（edit）以及删除（delete）部件的应用程序。我们可以创建一个可以让我们执行这些功能的HTTP API:

```sh
http://example.com/view_widgets
http://example.com/create_new_widget?name=Widgetizer
http://example.com/update_widget?id=123&name=Foo
http://example.com/delete_widget?id=123
```

当人们开始去实现他们自己的API接口时，问题就出现了。竟然没有一个标准的方法来命名URL，人们总是要参考API才得知它是如何运作的。一个API中可能命名一个URL为/view_widgets，但是另一个API可能就命名成/widgets/all.

不用担心！REST帮你搞定这些混乱！

## 什么是REST呢?

> REST是Representational State Transfer的缩写，它是由罗伊·菲尔丁(Roy Fielding)t提出的，是用来描述创建HTTP API的标准方法的，他发现这四种常用的行为（查看（view），创建(create)，编辑(edit)和删除(delete)）都可以直接映射到HTTP 中已实现的GET,POST,PUT和DELETE方法。

**HTTP 中的8中不同的方法：**

```sh
GET
POST
PUT
DELETE
OPTIONS
HEAD
TRACE
CONNECT
```

大多数情况下，当你在使用你的浏览器的点点看看的时候，其实只用到HTTP的GET方法。GET方法是在你向因特网请求资源的时候才会用到的。当你提交一个表单时，你就会经常用到POST方法来回传数据到网站上。至于其他的几种方法，某些浏览器可能根本就没有去完全实现它们。但是，如果是供我们使用的话，就没什么问题。问题是我们有很多要选择去帮助描述这四大行为的HTTP方法，我们将会用到那些已经知道如何去使用这些不同的HTTP方法的客户端类库。

### REST例子：

让我们来看下几个让API表述性状态转移化的例子，就用我们之前说的那几个部件来解释：

如果我们想要查看所有部件，URL将是这个样子：

```sh
GET http://example.com/widgets
```

用POST方法新建一个用来发出请求数据的部件：

```sh
POST http://example.com/widgets
Data:
    name = Foobar
```

用GET方法查看一个简单的部件，我们从指定的部件id中获取：

```sh
GET http://example.com/widgets/123
```

用PUT方法发送新数据来更新部件：

```sh
PUT http://example.com/widgets/123
Data:
    name = New name
    color = blue
```

用DELETE方法来删除部件：

```sh
DELETE http://example.com/widgets/123
```

**解剖REST URL：**

你可能已经注意的前面的几个例子，REST URL使用着一套一致的命名方法。当你跟API交互时，你几乎经常操作一些对象。在我们的例子中，我们讲的是部件。在REST中，我们称之为Resource。URL的第一部分经常是这个资源的复数形式：

```sh
/widgets
```

当我们参考收集的资源时（list all：列出所有 和add one：新增一个），这将会经常用到。当你用到一些特殊的资源的时候，你就会给URL增加一个id，这个URL在你想要“view”，“edit”和“delete”特殊资源的时候会被使用。

**嵌套资源：**

如果说，我们的部件有很多用户使用，URL的结构又将会是怎样的呢？

列出所有用户

```sh
GET /widgets/123/users
```

新增一个用户

```sh
POST /widgets/123/users
Data:
    name = Andrew
```

嵌套资源在URL里是完全兼容的，但是超过两层嵌套就不是很好的方法了。其实这根本不需要，因为你完全可以以ID的形式参考到那些嵌套资源，总比嵌套在父类中好。例如：

```sh
/widgets/123/users/456/sports/789
```

这可以替换为：

```sh
/users/456/sports/789
```

甚至可以替换成这样：

```sh
/sports/789
```

**HTTP 状态码：**

REST的另一重要部分就是为既定好请求的类型来响应正确的状态码。如果你对HTTP状态码陌生，以下是一个简易总结。当你请求HTTP时，服务器会响应一个状态码来判断你的请求是否成功，然后客户端应如何继续。以下是四种不同层次的状态码：

```sh
2xx = Success（成功）
3xx = Redirect（重定向）
4xx = User error（客户端错误）
5xx = Server error（服务器端错误）
```

以下是一些最重要的状态码：

**请求成功的状态码：**

```sh
200 – OK (默认的)
201 – Created（已创建）
202 – Accepted (已接受：常用语删除请求)
```

**客户端错误状态码：**

```sh
400 –请求出错（语法格式有误或服务器无法理解此请求）
401 – 未授权(需要登录)
404 – 找不到 (找不到所请求的文件或脚本)
405 – 不允许此方法(错误的 HTTP方法)
409 – 冲突 (IE尝试以PUT请求创建相同的资源时)
```

**API响应格式：**

当你请求HTTP时，你可以请求你想要接收的格式。例如，请求一个网页，你想以HTML的格式请求，或者如果你想要下载一张图片，返回格式应该是图片的格式。然而，响应请求格式是服务器的职责。

如今，JSON 已经快速发展成为REST API选择的格式，它有一个轻量级的、可读性又很高的语法，以致其很容易操作。所以，当使用我们API的用户按他们想要的格式发出请求和指定JSON时。

```sh
GET /widgets
Accept: application/json
```

我们的API将会以JSON的格式返回一批部件：

```json
[
  {
    id: 123,
    name: 'Simple Widget'
  },
  {
    id: 456,
    name: 'My other widget'
  }
]
```

要是用户请求一个我们没有实现的方法的格式时，我们又该怎么办呢？你大可以抛出一些错误的类型。但我建议你将JSON格式作为你的标准响应格式，因为这是开发者想要的格式。没理由去支持其他的格式，除非你已经有一个可支持的API。

**创建一个REST API：**

事实上，创建一个REST API是超出此教程范围的，因为它是有特定语言的。但我将以Ruby（一种为简单快捷的面向对象编程而创的脚本语言）的方式给出一个简易例子，它使用一个叫Sinatra的类库（不懂得可以自行百度）。

```ruby
require 'sinatra'
require 'JSON'
require 'widget' # our imaginary widget model

# list all
get '/widgets' do
  Widget.all.to_json
end

# view one
get '/widgets/:id' do
  widget = Widget.find(params[:id])
  return status 404 if widget.nil?
  widget.to_json
end

# create
post '/widgets' do
  widget = Widget.new(params['widget'])
  widget.save
  status 201
end

# update
put '/widgets/:id' do
  widget = Widget.find(params[:id])
  return status 404 if widget.nil?
  widget.update(params[:widget])
  widget.save
  status 202
end

delete '/widgets/:id' do
  widget = Widget.find(params[:id])
  return status 404 if widget.nil?
  widget.delete
  status 202
end
```

**API授权认证:**

在一般的网页应用中，认证操作是经常要接收用户名和密码的，然后在session中保存用户ID。用户的浏览器就会保存会话中的ID到cookie中。当用户在网站上访问需要认证授权的页面时，浏览器就会发送cookie，应用程序就会查找seesion会话中的ID（如果它没有失效的话），由于用户的ID保存在seesion中，用户就可以浏览页面了。

用这个API，就可以使用seesion会话保存用户记录,但这毕竟不是最好的方法。有时候，用户想直接访问API，或是用户想自己授权其他应用程序去访问这个API。

解决方法是在认证的基础上使用秘钥。用户输入用户名和密码以登录，应用程序就以一个特殊秘钥返回给用户以备后续之需。这个秘钥可以通入应用程序，以至于如果用户想要选择拒绝应用更进一步的接入时，可以撤回这个秘钥。

其实，网上已经有一个做上面这件事的很流行的标准方式，叫做OAuth（开放授权：是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），与以往的授权方式不同之处是OAUTH的授权不会使第三方触及到用户的帐号信息（如用户名与密码），即第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权，因此OAUTH是安全的。），特别的，标准第二版的OAuth。网上有很多非常好的实现OAuth的资源，所以我才说那是超出此教程范围的。如果你正在使用Ruby，这里有一些帮你解决大多数工作的很好的类库，比如[OmniAuth][1] 。

花了那么多时间给你们讲解这个教程，希望对你们有所帮助。

[原文出处][2]


  [1]: http://www.omniauth.org/
  [2]: http://codecloud.net/beginners-guide-to-creating-a-rest-api-6633.html