---
title: 跟上Java8 - Stream API快速入门
date: 2017-07-18
category: ["跟上Java8"]
tags: ["Java8", "stream", "flatmap", "filter", "collect"]
---

在前面我们简单介绍了`lambda`表达式，Java8旨在帮助程序员写出更好的代码，
其对核心类库的改进也是关键的一部分，`Stream`是Java8种处理集合的抽象概念，
它可以指定你希望对集合的操作，但是执行操作的时间交给具体实现来决定。

## 为什么需要Stream?

Java语言中集合是使用最多的API，几乎每个Java程序都会用到集合操作，
这里的Stream和IO中的Stream不同，它提供了对集合操作的增强，极大的提高了操作集合对象的便利性。

集合对于大多数编程任务而言都是基本的，为了解释集合是怎么工作，我们想象一下当下最火的外卖APP，
当我们点菜的时候需要按照**距离**、**价格**、**销量**等进行排序后筛选出自己满意的菜品。
你可能想选择距离自己最近的一家店铺点菜，尽管用集合可以完成这件事，但集合的操作远远算不上完美。

假如让你编写上面示例中的代码，你可能会写出如下：

```java
// 店铺属性
public class Property {
    String  name;
    // 距离，单位:米
    Integer distance;
    // 销量，月售
    Integer sales;
    // 价格，这里简单起见就写一个级别代表价格段
    Integer priceLevel;

    public Property(String name, int distance, int sales, int priceLevel) {
        this.name = name;
        this.distance = distance;
        this.sales = sales;
        this.priceLevel = priceLevel;
    }
    // getter setter 省略
}
```

我想要筛选距离我最近的店铺，你可能会写下这样的代码：

```java
public static void main(String[] args) {
    Property p1 = new Property("叫了个鸡", 1000, 500, 2);
    Property p2 = new Property("张三丰饺子馆", 2300, 1500, 3);
    Property p3 = new Property("永和大王", 580, 3000, 1);
    Property p4 = new Property("肯德基", 6000, 200, 4);

    List<Property> properties = Arrays.asList(p1, p2, p3, p4);

    Collections.sort(properties, (x, y) -> x.distance.compareTo(y.distance));

    String name = properties.get(0).name;
    System.out.println("距离我最近的店铺是:" + name);
}
```

这里也使用了部分`lambda`表达式，在Java8之前你可能写的更痛苦一些。
要是要处理大量元素又该怎么办呢？为了提高性能，你需要并行处理，并利用多核架构。
但写并行代码比用迭代器还要复杂，而且调试起来也够受的！

但`Stream`中操作这些东西当然是非常简单的，小试牛刀:

```java
// Stream操作
String name2 = properties.stream()
                .sorted(Comparator.comparingInt(x -> x.distance))
                .findFirst()
                .get().name;
System.out.println("距离我最近的店铺是:" + name);
```

新的API对所有的集合操作都提供了生成流操作的方法，写的代码也行云流水，我们非常简单的就筛选了离我最近的店铺。
在后面我们继续讲解`Stream`更多的特性和玩法。

## 外部迭代和内部迭代

当你处理集合时，通常会对它进行迭代，然后处理返回的每个元素。比如我想看看月销量大于1000的店铺个数。

### 使用for循环进行迭代

```java
int count = 0;
for (Property property : properties) {
    if(property.sales > 1000){
        count++;
    }
}
```

上面的操作是可行的，但是当每次迭代的时候你需要些很多重复的代码。将`for`循环修改为并行执行也非常困难，
需要修改每个`for`的实现。

从集合背后的原理来看，`for`循环封装了迭代的语法糖，首先调用`iterator`方法，产生一个`Iterator`对象，
然后控制整个迭代，这就是**外部迭代**。迭代的过程通过调用`Iterator`对象的`hasNext`和`next`方法完成。

### 使用迭代器进行计算

```java
int count = 0;
Iterator<Property> iterator = properties.iterator();
while(iterator.hasNext()){
    Property property = iterator.next();
    if(property.sales > 1000){
        count++;
    }
}
```

而迭代器也是有问题的。它很难抽象出**未知的不能操作**；此外它本质上还是串行化的操作，总体来看使用
`for`循环会将行为和方法混为一谈。

另一种办法是使用内部迭代完成，`properties.stream()`该方法返回一个`Stream`而不是迭代器。

### 使用内部迭代进行计算

```java
long count2 = properties.stream()
                .filter(p -> p.sales > 1000)
                .count();
```

上述代码是通过`Stream API`完成的，我们可以把它理解为2个步骤：

1. 找出所有销量大于1000的店铺
2. 计算出店铺个数

为了找出销量大于1000的店铺，需要先做一次过滤：`filter`，你可以看看这个方法的入参就是前面讲到的`Predicate`断言型函数式接口，
测试一个函数完成后，返回值为`boolean`。
由于`Stream API`的风格，我们没有改变集合的内容，而是描述了`Stream`的内容，最终调用`count()`方法计算出`Stream`
里包含了多少个过滤之后的对象，返回值为`long`。

## 创建Stream

你已经知道Java8种在`Collection`接口添加了`Stream`方法，可以将任何集合转换成一个`Stream`。
如果你操作的是一个数组可以使用`Stream.of(1, 2, 3)`方法将它转换为一个流。

也许有人知道JDK7中添加了一些类库如`Files.readAllLines(Paths.get("/home/biezhi/a.txt"))`这样的读取文件行方法。
`List`作为`Collection`的子类拥有转换流的方法，那么我们读取这个文本文件到一个字符串变量中将变得更简洁：

```java
String content = Files.readAllLines(Paths.get("/home/biezhi/a.txt")).stream()
            .collect(Collectors.joining("\n"));
```

这里的`collect`是后面要讲解的**收集器**，对`Stream`进行了处理后得到一个文本文件的内容。

JDK8也为我们提供了一些便捷的`Stream`相关类库:

<p>
    {% img image-bubble /static/img/article/java8-stream-class.png 320 500 java8提供的Stream类库 %}
</p>

创建一个流是很简单的，下面我们试试用创建好的`Stream`做一些操作吧。

## 流操作

`java.util.stream.Stream`中定义了许多流操作的方法，为了更好的理解`Stream API`掌握它常用的操作非常重要。
流的操作其实可以分为两类：**处理操作**、**聚合操作**。

- 处理操作：诸如`filter`、`map`等处理操作将`Stream`一层一层的进行抽离，返回一个流给下一层使用。
- 聚合操作：从最后一次流中生成一个结果给调用方，`foreach`只做处理不做返回。

### filter

`filter`看名字也知道是过滤的意思，我们通常在筛选数据的时候用到，频率非常高。
`filter`方法的参数是`Predicate<T> predicate`即一个从`T`到boolean的函数。

<p>
    {% img image-bubble /static/img/article/java8-filter.png 300 200 %}
</p>

**筛选出距离我在1000米内的店铺**

```java
properties.stream()
            .filter(p -> p.distance < 1000)
```

**筛选出名称大于5个字的店铺**

```java
properties.stream()
            .filter(p -> p.name.length() > 5);
```

### map

有时候我们需要将流中处理的数据类型进行转换，这时候就可以使用`map`方法来完成，将流中的值转换为一个新的流。

<p>
    {% img image-bubble /static/img/article/java8-map.png 300 200 %}
</p>

**列出所有店铺的名称**

```java
properties.stream()
            .map(p -> p.name);
```

传给`map`的`lambda`表达式接收一个`Property`类型的参数，返回一个`String`。
参数和返回值不必属于同一种类型，但是`lambda`表达式必须是`Function`接口的一个实例。

### flatMap

有时候我们会遇到提取子流的操作，这种情况用的不多但是遇到`flatMap`将变得更容易处理。

<p>
    {% img image-bubble /static/img/article/java8-flatmap.png 300 200 %}
</p>

例如我们有一个`List<List<String>>`结构的数据：

```java
List<List<String>> lists = new ArrayList<>();
        lists.add(Arrays.asList("apple", "click"));
        lists.add(Arrays.asList("boss", "dig", "qq", "vivo"));
        lists.add(Arrays.asList("c#", "biezhi"));
```

要做的操作是获取这些数据中长度大于2的单词个数

```javascript
lists.stream()
        .flatMap(Collection::stream)
        .filter(str -> str.length() > 2)
        .count();
```

在不使用`flatMap`前你可能需要做2次`for`循环。这里调用了`List`的`stream`方法将每个列表转换成`Stream`对象，
其他的就和之前的操作一样。

## max和min

`Stream`中常用的操作之一是求最大值和最小值，`Stream API` 中的`max`和`min`操作足以解决这一问题。

我们需要筛选出价格最低的店铺:

```java
Property property = properties.stream()
            .max(Comparator.comparingInt(p -> p.priceLevel))
            .get();
```

查找`Stream`中的最大或最小元素，首先要考虑的是用什么作为排序的指标。
以查找价格最低的店铺为例，排序的指标就是**店铺的价格等级**。

为了让`Stream`对象按照价格等级进行排序，需要传给它一个`Comparator`对象。
Java8提供了一个新的静态方法`comparingInt`，使用它可以方便地实现一个比较器。
放在以前，我们需要比较两个对象的某项属性的值，现在只需要提供一个存取方法就够了。

## 聚合操作



## 收集结果

## 并行数据处理