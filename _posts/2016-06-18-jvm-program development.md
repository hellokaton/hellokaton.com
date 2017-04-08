---
layout: post
title: 聊聊JVM - 从程序开发说起
categories: jvm
tags: jvm
---

我们都知道java是一种面向对象、跨平台、健壮安全balabala的程序语言。这将是jvm入门的一系列文章的开头，编写一个 `Hello World` 程序是这样的：

```java
public class Hello {
    public static void main(String[] args){
        System.out.println("Hello JVM!");
    }
}
```

<!-- more -->

在操作系统上使用 `javac` 命令进行编译后：

![Alt text](https://i.imgur.com/0xu1csP.png)

执行结果：

```bash
→ java Hello
Hello JVM!
```

java的一大特色就是"write once, run anywhere" 即 “一次编译，到处运行”。就是说你不用专门为每个平台写一份代码，你写的Java程序在任何平台都能跑起来。

它的实现原理是在系统层面上又增加了一层虚拟机（Java Virtual Machine，简称JVM），且为每个平台都定制了对应的虚拟机。然后Java程序是在虚拟机上跑的，因此平台无关。

Java的运行流程是：程序员写了源代码（Source Code，.java后缀，跨平台），然后经过编译器编译成字节码（Byte Code，.class后缀，二进制文件，跨平台），字节码是所有虚拟机都能理解的中间文件。然后交给虚拟机（不跨平台，每个平台都有对应的虚拟机）去运行。
所以对“write once, run anywhere”更准确的理解是，“一次编译，到处装虚拟机，所以到处运行”。

java程序执行流程如图

![Java程序执行流程](https://i.imgur.com/g52hveh.png)

## java程序运行原理

上面我们写了一个最简单的java程序，这是每个java程序员入门的第一行代码，那么在这行代码的背后到底发生了什么呢？我们来探究一下。

![java运行原理](https://i.imgur.com/1cltaEJ.png)

Java的一大特色就是“write once, run anywhere”即“一次编译，到处运行”。就是说你不用专门为每个平台写一份代码，你写的Java程序在任何平台都能跑起来。

它的实现原理是在系统层面上又增加了一层虚拟机（Java Virtual Machine，简称JVM），且为每个平台都定制了对应的虚拟机。然后Java程序是在虚拟机上跑的，因此平台无关。

Java的运行流程是：程序员写了源代码（Source Code，.java后缀，跨平台），然后经过编译器编译成字节码（Byte Code，.class后缀，二进制文件，跨平台），字节码是所有虚拟机都能理解的中间文件。然后交给虚拟机（不跨平台，每个平台都有对应的虚拟机）去运行。
所以对“write once, run anywhere”更准确的理解是，“一次编译，到处装虚拟机，所以到处运行”。

## java编译运行过程

Java代码编译是由Java源码编译器来完成，流程图如下所示：

![java编译过程](https://i.imgur.com/gFtin2q.jpg)

图片来自[http://blog.csdn.net/cutesource/article/details/5904542](http://blog.csdn.net/cutesource/article/details/5904542)

Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：

![jvm执行引擎](https://i.imgur.com/TauXjos.jpg)

图片来自[http://blog.csdn.net/cutesource/article/details/5904542](http://blog.csdn.net/cutesource/article/details/5904542)

我们能看出java的整个编译过程是非常繁琐的，下面用一个例子来介绍一下这个过程。

java程序从创建到运行的步骤如下图：

![java程序编译运行过程](https://i.imgur.com/4gGJ4Dn.png)

图片来自[http://www.borjournals.com/a/index.php/jecas/article/viewFile/289/866](http://www.borjournals.com/a/index.php/jecas/article/viewFile/289/866)

创建2个java文件：

```java
public class Person {
    private String name;
    private int age;
    public Person(String name){
        this.name = name;
    }
    
    public void printName(){
	    System.out.println("Person["+ this.name +"]");
    }
}
```

```java
public class MainApp {
    public static void main(String[] args){
		Person p = new Person("李小龙");
		p.printName();
    }
}
```

ok，非常简单的2个java文件，先编译 `Person` 类，再编译 `MainApp`，生成2个字节码文件。

**第一步(编译)**: 创建完源文件之后，程序会先被编译为.class文件。Java编译一个类时，如果这个类所依赖的类还没有被编译，编译器就会先编译这个被依赖的类，然后引用，否则直接引用，这个有点象make。如果java编译器在指定目录下找不到该类所其依赖的类的.class文件或者.java源文件的话，编译器话报“cant find symbol”的错误。

编译后的字节码文件格式主要分为两部分：**常量池和方法字节码**。常量池记录的是代码出现过的所有token(类名，成员变量名等等)以及符号引用（方法引用，成员变量引用等等）；方法字节码放的是类中各个方法的字节码。下面是`MainApp.class`通过 `javap -verbose MainApp` 的结果，我们可以清楚看到`.class`文件的结构：

```bash
Classfile /Users/biezhi/workspace/java/jvm/MainApp.class
  Last modified 2016-6-18; size 364 bytes
  MD5 checksum e09fde0fa30f1e35b26e588df43af73b
  Compiled from "MainApp.java"
public class MainApp
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
  
// 常量池信息
Constant pool:
   #1 = Methodref          #7.#16         // java/lang/Object."<init>":()V
   #2 = Class              #17            // Person
   #3 = String             #18            // 李小龙
   #4 = Methodref          #2.#19         // Person."<init>":(Ljava/lang/String;)V
   #5 = Methodref          #2.#20         // Person.printName:()V
   #6 = Class              #21            // MainApp
   #7 = Class              #22            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               main
  #13 = Utf8               ([Ljava/lang/String;)V
  #14 = Utf8               SourceFile
  #15 = Utf8               MainApp.java
  #16 = NameAndType        #8:#9          // "<init>":()V
  #17 = Utf8               Person
  #18 = Utf8               李小龙
  #19 = NameAndType        #8:#23         // "<init>":(Ljava/lang/String;)V
  #20 = NameAndType        #24:#9         // printName:()V
  #21 = Utf8               MainApp
  #22 = Utf8               java/lang/Object
  #23 = Utf8               (Ljava/lang/String;)V
  #24 = Utf8               printName
  
// MainApp类方法字节码信息
{
  public MainApp();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class Person
         3: dup
         4: ldc           #3                  // String 李小龙
         6: invokespecial #4                  // Method Person."<init>":(Ljava/lang/String;)V
         9: astore_1
        10: aload_1
        11: invokevirtual #5                  // Method Person.printName:()V
        14: return
      LineNumberTable:
        line 3: 0
        line 4: 10
        line 5: 14
}
SourceFile: "MainApp.java"
```

**第二步（运行）**：我是mac系统下jdk1.8的环境，这里给出了非常详细的类结构，java类运行的过程大概可分为两个过程：
1. 类的加载
2. 类的执行

> JVM在程序第一次主动使用类的时候，才会去加载该类，并非在一开始就把一个程序就所有的类都加载到内存中，而是到不得不用的时候才把它加载进来，且只加载一次。

下面是程序运行的详细步骤：

1. 在编译好java程序得到`MainApp.class`文件后，在命令行上敲`java MainApp`。系统就会启动一个jvm进程，jvm进程从classpath路径中找到一个名为`MainApp.class`的二进制文件，将`MainApp`的类信息加载到**运行时数据区**的方法区内，这个过程叫做`MainApp`类的加载
2. JVM找到`MainApp`的主函数入口，开始执行main函数
3. main函数的第一条命令是`Person p = new Person("李小龙");`就是让JVM创建一个`Person`对象，但是这时候**方法区**中没有`Person`类的信息，所以JVM马上加载`Person`类，把`Person`类的类型信息放到方法区中
4. 加载完`Person`类之后，Java虚拟机做的第一件事情就是在堆区中为一个新的Animal实例分配内存, 然后调用构造函数初始化`Person`实例，这个`Person`实例持有着指向方法区的`Person`类的类型信息（其中包含有方法表，java动态绑定的底层实现）的引用
5. 当调用`p.printName()`的时候，JVM根据`p`引用找到`Person`对象，然后根据`Person`对象持有的引用定位到方法区中`Person`类的类型信息的方法表，获得`printName()`函数的字节码的地址
6. 执行`printName()`函数

参考资料：

- [http://blog.csdn.net/cutesource/article/details/5904542](http://blog.csdn.net/cutesource/article/details/5904542)
- [http://www.borjournals.com/a/index.php/jecas/article/viewFile/289/866](http://www.borjournals.com/a/index.php/jecas/article/viewFile/289/866)
- [http://coolshell.cn/articles/9229.html](http://coolshell.cn/articles/9229.html)
