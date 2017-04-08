---
layout: post
title: Python读写zip压缩文件
categories: python
tags: python zip
---

Python自带模块zipfile可以完成zip压缩文件的读写，而且使用非常方便，下面我们就来演示一下Python读写zip文件。

<!-- more -->

### Python读zip文件

下面的代码给出了用Python读取zip文件，打印出压缩文件里面所有的文件，并读取压缩文件中的第一个文件。

```python
import zipfile
 
z = zipfile.ZipFile("zipfile.zip", "r")
 
#打印zip文件中的文件列表
for filename in z.namelist(  ):
    print 'File:', filename
 
#读取zip文件中的第一个文件
first_file_name = z.namelist()[0]
content = z.read(first_file_name)
print first_file_name
print content
```

### Python写/创建zip文件

Python写Zip文件主要用到ZipFile的write函数。

```python
import zipfile
 
z = zipfile.ZipFile('test.zip', 'w', zipfile.ZIP_DEFLATED)
z.write('test.html')
z.close( ) 
```

在创建ZipFile实例的时候，有2点药注意：

1. 要用'w'或'a'模式，用可写的方式打开zip文件
2. 压缩模式有ZIP_STORED 和 ZIP_DEFLATED，ZIP_STORED只是存储模式，不会对文件进行压缩，这个是默认值，如果你需要对文件进行压缩，必须使用ZIP_DEFLATED模式。
