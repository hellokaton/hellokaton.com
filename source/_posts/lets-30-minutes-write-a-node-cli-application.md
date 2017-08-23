---
title: 30分钟使用Node实现一个命令行程序
date: 2017-08-22
category: [nodejs]
tags: ["nodejs", "terminal"]
---

让我很无奈的是使用Java编写命令行程序是比较麻烦的，好在NodeJs干这事很方便，
在接下来的30分钟里我将教你编写一个有趣的终端程序并将它发布到npm仓库中，赶紧GET吧~

<!-- more -->

{% image /static/img/article/node-cli.png 650 380 lowb终端程序 %}

我实在想不到起什么名字了，就叫 `lowb` 吧。。。我们实现好的程序是这样的：

<script type="text/javascript" src="https://asciinema.org/a/132942.js" id="asciicast-132942" async></script>

我把源码放在：https://github.com/biezhi/lowb
如何录制终端命令，我写了一篇文章 [教你使用asciinema录制命令行操作](http://biezhi.me/2017/08/08/teach-you-how-to-use-asciinema.html)

在开始之前请务必确认你安装了Node的环境，我目前使用的NodeJs环境是 `v6.10.3`。

## 创建项目结构

```bash
mkdir lowb && cd lowb && npm init
```

根据提示初始化你的项目，如果你不懂怎么填写一路回车也是可以的（终端会提示你输入一些项目信息）

**创建一个二进制目录**

```bash
mkdir bin
```

在 `lowb` 根目录下创建一个 `bin` 目录，我们将可执行文件存放在这里，此时你的项目结构是这样的：

```bash
⬢  lowb
.
├── bin
└── package.json

1 directory, 1 file
```

## 光杆司令的做法

原理是解析Node中的进程对象 [process.argv](http://nodejs.cn/api/process.html#process_process_argv)

### 首先在 `bin` 目录下创建一个文件 `lowb.js` 作为我们的运行程序

```js
#!/usr/bin/env node

var fs = require("fs"), path = process.cwd();
var appInfo = require('../package.json');

function app(obj) {
    if(obj[0] === '-v' || obj[0] === '--version'){
        console.log('  version is ' + appInfo.version);
    } else if(obj[0] === '-h' || obj[0] === '--help'){
        console.log('Useage:');
        console.log('  -v --version [show version]');
    } else{
        fs.readdir(path, function(err, files){
            if(err){
                return console.log(err);
            }
            for(var i = 0; i < files.length; i += 1){
                console.log(files[i]);
            }
        });
    }
};

//获取除第一个命令以后的参数，使用空格拆分
app(process.argv.slice(2));
```

这段代码中非常简单，我们首先引入了 `fs` 模块和 `package.json` 的变量（用于获取版本号）。
然后编写了一个函数进行调用，里面只实现了一个 `-v` 和 `-h` 的命令。

> `process.argv` 是一个数组，第一个元素返回node执行路径，第二个元素是当前执行文件的路径，
> 从第三个开始是运行时带的参数

### 指定执行脚本

修改一下 `package.json`

```js
{
  "name": "lowb",
  "version": "1.0.0",
  "description": "lowb项目，在命令行下输出名言、段子、诗歌的小玩意~",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/biezhi/lowb.git"
  },
  "keywords": [
    "lowb"
  ],
  "bin": {
    "lowb": "bin/lowb.js"
  },
  "author": "biezhi <biezhi.me@gmail.com>",
  "license": "MIT"
}
```

注意添加了 `bin` 这个参数，将 `lowb` 这个命令映射到 `bin/lowb.js` 这个文件，
我们打包安装后调用 `lowb` 命令会执行 `bin/lowb.js` 中的内容。

### 安装到本地

```bash
sudo npm install . -g
```

在 `lowb` 根目录下执行如上命令，你将看到类似如下的输出

```bash
⬢  lowb  sudo npm install . -g
Password:
/Users/biezhi/.nvm/versions/node/v6.10.3/bin/lowb
    -> /Users/biezhi/.nvm/versions/node/v6.10.3/lib/node_modules/lowb/bin/lowb.js
/Users/biezhi/.nvm/versions/node/v6.10.3/lib
```

此时已经将命令安装到本地了，我们可以试试在终端下运行了：

```bash
⬢  lowb  lowb
bin
package.json
⬢  lowb  lowb -v
version is 1.0.0
⬢  lowb  lowb -h
Useage:
  -v --version [show version]
⬢  lowb  lowb --help
bin
package.json
```

## 试试Commander.js

前面这种做法当然是可以完成一个命令行程序的，但这样的做法就像是在刀耕火种的时代，而且功能有限；
在前端界赫赫有名的工程师 [tj](https://github.com/tj) 写了一个库 [commander.js](https://github.com/tj/commander.js)
就是帮助我们简化命令行程序开发。

它是受Ruby中的一个库 [commander](https://github.com/commander-rb/commander) 而诞生，是一款轻量级表现力强大的命令行框架。

我们来使用这个工具完成今天要做的小玩意。

### 安装Commander.js

```bash
npm install commander --save
```

### 快速入门

```js
var program = require('commander');

program
  .version('0.1.0')
  .option('-p, --peppers', 'Add peppers')
  .option('-P, --pineapple', 'Add pineapple')
  .option('-b, --bbq-sauce', 'Add bbq sauce')
  .option('-c, --cheese [type]', 'Add the specified type of cheese [marble]', 'marble')
  .parse(process.argv);

console.log('you ordered a pizza with:');
if (program.peppers) console.log('  - peppers');
if (program.pineapple) console.log('  - pineapple');
if (program.bbqSauce) console.log('  - bbq');
console.log('  - %s cheese', program.cheese);
```

这是官方README中的一段代码，我们写一个JS运行一下 `cmd -h` 看看

```bash
Usage: cmd [options]


  Options:

    -V, --version        output the version number
    -p, --peppers        Add peppers
    -P, --pineapple      Add pineapple
    -b, --bbq-sauce      Add bbq sauce
    -c, --cheese [type]  Add the specified type of cheese [marble]
    -h, --help           output usage information
```

**常用api**

`commander.js` 中命令行有两种可变性，`option`：选项，`command`：命令。

- 通过`option`设置的选项可以通过`program.chdir`或者`program.noTests`来访问。
- 通过`command`设置的命令通常在`action`回调中处理逻辑。

本文中没有用到`command`就不详述了。

### 开始干它一票吧

我们希望做出来的使用效果是这样的:

```bash
lowb --help

  Usage: lowb [options]

  Options:

    -V, --version    output the version number
    -i, --index <n>  ascii art index, default is random
    -t, --type <value>  quotes/jokes/tang/song
    -h, --help       output usage information
```

- `-V, --version`: 输出程序的版本号
- `-i, --index <n>`: ascii动物的索引，默认是随机的
- `-t, --type <value>`: 输出文本的类型，名言、段子、唐诗、宋词，默认是名言
- `-h, --help`: 帮助信息

引入 `commander`

```js
var cmd     = require('commander');
```

编写命令处理的代码

```js
cmd.version(appInfo.version)
    .option('-i, --index <n>', 'ascii art index, default is random', -1, parseInt)
    .option('-t, --type <value>', '[quotes|jokes|tang|song]', 'quotes', /^(quotes|jokes|tang|song)$/i)
    .on('--help', function(){
        console.log('\t' + appInfo.repository.url);
    }).parse(process.argv);
```

`option` 的常用API

- 第一个参数是选项定义，分为短定义和长定义，用|，,，连接，参数可以用<>或者[]修饰，前者意为必须参数，后者意为可选参数
- 第二个参数为选项描述
- 第三个参数为选项参数默认值，可选

### 准备原材料

什么原材料？我们需要输出ascii的动物图像和一些名言、段子等文本，这里数据我就存放在 `data` 目录下。
当然做的高级点你可以用爬虫~ 本文引砖抛玉了

我准备了 `animals.txt` 里面存放的是ascii的动物图像，像这样：

{% image https://i.loli.net/2017/08/22/599c2b7050c5c.png 450 752 lowb终端程序 %}

如果觉得这些你不喜欢可以上[这里](http://www.asciiworld.com/-Animals-.html)看看还有更多的小动物任你把玩。

还有 [段子](https://github.com/biezhi/lowb/blob/master/data/jokes.txt)，[名言](https://github.com/biezhi/lowb/blob/master/data/quotes.txt)，
[唐诗](https://github.com/biezhi/lowb/blob/master/data/tang300.txt)，[宋词](https://github.com/biezhi/lowb/blob/master/data/song100.txt) 的数据

将这5个文本数据存放在 `data` 目录下，使用JS变量存储一下

```js
var fs      = require('fs');
var path    = require('path');

var animals = fs.readFileSync(path.join(__dirname, '../data/animals.txt')).toString()
                .split('===============++++SEPERATOR++++====================\n');

var jokes  = fs.readFileSync(path.join(__dirname, '../data/jokes.txt')).toString().split('%\n');
var quotes  = fs.readFileSync(path.join(__dirname, '../data/quotes.txt')).toString().split('%\n');
var tang300 = fs.readFileSync(path.join(__dirname, '../data/tang300.txt')).toString().split('%\n');
var song100 = fs.readFileSync(path.join(__dirname, '../data/song100.txt')).toString().split('%\n');
```

### 读取数据

编写2个函数，一个用于产生随机的ascii动物文本，一个用于返回名言或段子

```js
/**
 * 返回一个随机的动物ascii
 *
 * @returns {*}
 */
function randomAnimal() {
    return animals[Math.floor(Math.random() * animals.length)];
}

/**
 * 根据类型返回名言或段子
 *
 * @param type
 * @returns {string}
 */
function prefix(type) {
    switch (type) {
        case 'quotes':
            return quotes[Math.floor(Math.random() * quotes.length)];
        case 'jokes':
            return jokes[Math.floor(Math.random() * jokes.length)];
        case 'tang':
            return tang300[Math.floor(Math.random() * tang300.length)];
        case 'song':
            return song100[Math.floor(Math.random() * song100.length)];
        default:
            return tang300[Math.floor(Math.random() * tang300.length)];
    }
}
```

然后在命令解析的下面开始调用他们吧~

```js
var animal = cmd.index === -1 ? randomAnimal() : animals[cmd.index];
console.log(prefix(cmd.type));
console.log(animal);
```

这里的逻辑非常简单，如果没有指定动物索引则随机获取一个小动物，否则取指定的。
然后输出名言/段子，输出一个小动物~ 大功告成

### 来试试吧

程序编写ok后我们安装到本地，然后执行 `lowb -t jokes` 试试看？

```bash

    阿里小米皆自主，百度排名最公平；
    京东全网最低价，当当爱国很理性；
    用户体验看新浪，网易从来少愤青；
    豆瓣从来不约炮，人人分享高水平；
    从不抄袭数腾讯， 开放安全三六零。

                                  ____
           ,.-''''-,__,..---'''```    ``''-.
          //      '   `.                    `,
         7;                         )         .
        Y    \  /                  /           L,
        : \. \\|                 ,`            | `'.
    ,.-'^, \\``',    (           ;             ;    `,
   //`_),.\|\)_ .\   /          ,A          ._,^      \
   L\)  ,+`[  e\ \.-`''--......-__`.  _,       `\.     )Y
   _,--`   \   )`.`,           // ````          / )_.-' |
  //,/`_)'` `''-. `/    _,.......----------'"""'``      /
  \\)\)          `"   +`  ________                   _,`
   `` `             ,`,'``         ```````'"""""""'``
                    |7  sk
                     \_,
```

默认输出的是名言，我加了`-t`参数指定来个段子。

## 发布到NPM仓库

我们希望所有人都可以下载到自己写的程序就需要将它发布到npm的仓库中，
发布前需要注册npm账号，顺手绑定一下你的github账号吧。注册ok后在项目根目录下执行 `sudo npm publish`
会提示你输入一下npm的账号和密码验证，完了就会推送到 npmjs.org 了，你可以搜到自己的程序。

像安装其他程序一样安装 `sudo npm install lowb -g`（记得把本地安装的卸载了）

通过这篇文章相信你也可以编写一个终端程序玩了~ (๑•̀ㅂ•́) ✧加油