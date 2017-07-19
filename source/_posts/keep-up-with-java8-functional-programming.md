---
title: 跟上Java8 - 函数式编程
date: 2017-07-19
category: ["跟上Java8"]
tags: ["Java8", "函数式"]
---

{% image /static/img/article/java8-function-programing.jpg 600 400 函数式编程 %}

在前面的章节我们快速学习了`lambda`和`Stream`，本章节中我们来回顾和理解函数式编程的思想。
我们不断的提及`函数式`这个名词，它指的是lambda吗？如果是这样，采用函数式编程能为你带来什么好处呢?

<!-- more -->

## 函数式的思考

### 命令式编程

一般我们实现一个系统有两种思考方式，一种专注于如何实现，比如下厨做菜，通常按照自己熟悉的烹饪方法：首先洗菜，
然后切菜，热油，下菜，然后…… 这看起来像是一系列的命令合集。对于这种"如何做"式的编程风格我们称之为**命令式编程**，
它的特点非常像工厂的流水线、计算机的指令处理，都是串行化、命令式的。

```java
CookingTask cookingTask = new CookingTask();
cookingTask.wash();
cookingTask.cut();
cookingTask.deepFry();
cookingTask.fried();
...
```

### 声明式编程

还有一种方式你关注的是要做什么，我们如果用`lambda`和函数式来解决上述问题应该是这样的：

```java
public class CookingDemo {

    public void doTask(String material, Consumer<String> consumer) {
        consumer.accept(material);
    }

    public static void main(String[] args) {
        CookingDemo cookingDemo = new CookingDemo();
        cookingDemo.doTask("蔬菜", material -> System.out.println("清洗" + material));
        cookingDemo.doTask("蔬菜", material -> System.out.println(material + "切片"));
        cookingDemo.doTask("食用油", material -> System.out.println(material + "烧热"));
        cookingDemo.doTask("", material -> System.out.println("炒菜"));
    }
}
```

这里我们将烹饪的实现细节交给了函数库，它最大的优势在于你读起来就像是在问题陈述，采用这种方式我们很快可以理解它的功能，
当你在烹饪流程中添加其他步骤也变得非常简单，你只需要调用`doTask`方法将材料传递进去处理，比如在食用油烧热前我要打个鸡蛋

```java
cookingDemo.doTask("鸡蛋", material -> System.out.println(material + "打碎搅拌均匀"));
```

而不用再编写一个处理鸡蛋的方法。

## 什么是函数式编程

对于“什么是函数式编程”这一问题最简化的回答是“它是一种使用函数进行编程的方式”。
每个人的理解都是不同的，其核心是：**在思考问题时，使用不可变值和函数，函数对一个值进行处理，映射成另一个值。**

不同的语言社区往往对各自语言中的特性孤芳自赏。现在谈Java程序员如何定义函数式编程还为时尚早，
但是，这根本不重要！我们关心的是如何写出好代码，而不是符合函数式编程风格的代码。

我们想象一下设计一个函数，输入一个字符串类型和布尔类型参数，输出一个整形参数。

```java
int pos = 0;
public Integer foo(String str, boolea flag){
    if(flag && null != str){
        pos++;
    }
    return pos;
}
```

这个例子有输入也有输出，同时每次调用也可能会更行外部的变量值，这样的函数我们称之为是有**副作用**的函数。

在函数式编程的上下文中，一个“函数”对应于一个数学函数：它接受零个或多个参数，生成一个或多个结果，并且不会有任何副作用。
你可以把它看成一个黑盒，它接收输入并产生一些输出，像下面的函数

```java
public Integer foo(String str, boolea flag){
    if(flag && null != str){
        return 1;
    }
    return 0;
}
```

这种类型的函数和你在Java编程语言中见到的函数之间的区别是非常重要的（我们无法想象，log或者sin这样的数学函数会有副作用）。
尤其是，使用同样的参数调用数学函数，它所返回的结果一定是相同的。这里，我们暂时不考虑`Random.nextInt`这样的方法，

## 函数的副作用

当谈论“函数式”时，我们想说的其实是“像数学函数那样——没有副作用”。由此，编程上的一些精妙问题随之而来。
我们的意思是，每个函数都只能使用函数和像`if-then-else`这样的数学思想来构建吗？
或者，我们也允许函数内部执行一些非函数式的操作，只要这些操作的结果不会暴露给系统中的其他部分？
换句话说，如果程序有一定的副作用，不过该副作用不会为其他的调用者感知，是否我们能假设这种副作用不存在呢？
调用者不需要知道，或者完全不在意这些副作用，因为这对它完全没有影响。

当我们希望能界定这二者之间的区别时，我们将第一种称为纯粹的函数式编程，后者称为函数式编程。

在编程实战中我们很难用Java语言以纯粹的函数式来完成一个程序的，因为很多老的代码包括标准库的函数都是有副作用的
（调用`Scanner.nextLine`就有副作用，它会从一个文件中读取一行， 通常情况两次调用的结果完全不同）。你希望为你的系统
编写接近纯函数式的实现，需要确保你的代码没有副作用。假设这样一个函数或者方法，它没有副作用，进入方法体执行时会对一个字段的值加一，
退出方法体之前会对该字段减一。对一个单线程的程序而言，这个方法是没有副作用的，可以看作函数式的实现。

我们构建函数式的准则是，被称为“函数式”的函数或方法都只能修改局部变量，除此之外，它引用的对象都应该是`final`的。
所有的引用类型字段都指向不可变对象。