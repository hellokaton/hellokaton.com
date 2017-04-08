---
layout: post
title: npm常用命令
categories: nodejs
tags: nodejs npm
---

`npm install <name>` 安装nodejs的依赖包

例如`npm install express` 就会默认安装express的最新版本，也可以通过在后面加版本号的方式安装指定版本，如npm install express@3.0.6

<!-- more -->

`npm install <name> -g`  将包安装到全局环境中

但是代码中，直接通过require()的方式是没有办法调用全局安装的包的。全局的安装是供命令行使用的，就好像全局安装了vmarket后，就可以在命令行中直接运行vm命令

`npm install <name> --save`  安装的同时，将信息写入package.json中
项目路径中如果有package.json文件时，直接使用npm install方法就可以根据dependencies配置安装所有的依赖包

这样代码提交到github时，就不用提交node_modules这个文件夹了。

npm init  会引导你创建一个package.json文件，包括名称、版本、作者这些信息等

`npm remove <name>` 移除

`npm update <name>` 更新

`npm ls` 列出当前安装的了所有包

`npm root` 查看当前包的安装路径

`npm root -g`  查看全局的包的安装路径

`npm help`  帮助，如果要单独查看install命令的帮助，可以使用的npm help install

[https://npmjs.org/doc/](https://npmjs.org/doc/)
