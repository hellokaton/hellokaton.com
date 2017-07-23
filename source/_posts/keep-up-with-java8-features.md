---
title: 跟上Java8 - 你忽略了的新特性
date: 2017-07-21
category: ["跟上Java8"]
tags: ["Java8", "features"]
---

虽然我们开始了Java8的旅程，但是很多人直接从java6上手了java8，
也许有一些JDK7的特性你还不知道，在本章节中带你回顾一下我们忘记了的那些特性。
尽管我们不能讲所有特性都讲一遍，挑出常用的核心特性拎出来一起学习。

<!-- more -->

{% image /static/img/article/java8-new-features.jpg 550 350 新特新 %}

## 异常改进

### try-with-resources

这个特性是在JDK7种出现的，我们在之前操作一个流对象的时候大概是这样的：

```java
try {
    // 使用流对象
    stream.read();
    stream.write();
} catch(Exception e){
    // 处理异常
} finally {
    // 关闭流资源
    if(stream != null){
        stream.close();
    }
}
```

这样无疑有些繁琐，而且`finally`块还有可能抛出异常。在JDK7种提出了`try-with-resources`机制，
它规定你操作的类只要是实现了`AutoCloseable`接口就可以在`try`语句块退出的时候自动调用`close`
方法关闭流资料。

```java
public static void tryWithResources() throws IOException {
    try( InputStream ins = new FileInputStream("/home/biezhi/a.txt") ){
        char charStr = (char) ins.read();
        System.out.print(charStr);
    }
}
```

**使用多个资源**

```java
try ( InputStream is  = new FileInputStream("/home/biezhi/a.txt");
      OutputStream os = new FileOutputStream("/home/biezhi/b.txt")
) {
    char charStr = (char) is.read();
    os.write(charStr);
}
```

当然如果你使用的是非标准库的类也可以自定义`AutoCloseable`，只要实现其`close`方法即可。

### catch多个Exception

当我们在操作一个对象的时候，有时候它会抛出多个异常，像这样：

```java
try {
    Thread.sleep(20000);
    FileInputStream fis = new FileInputStream("/a/b.txt");
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
```

这样代码写起来要捕获很多异常，不是很优雅，JDK7种允许你捕获多个异常:

```java
try {
    Thread.sleep(20000);
    FileInputStream fis = new FileInputStream("/a/b.txt");
} catch (InterruptedException | IOException e) {
    e.printStackTrace();
}
```

并且`catch`语句后面的异常参数是`final`的，不可以再修改/复制。

### 处理反射异常

使用过反射的同学可能知道我们有时候操作反射方法的时候会抛出很多不相关的检查异常，例如：

```java
try {
    Class<?> clazz = Class.forName("com.biezhi.apple.User");
    clazz.getMethods()[0].invoke(object);
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

尽管你可以使用`catch`多个异常的方法将上述异常都捕获，但这也让人感到痛苦。
JDK7修复了这个缺陷，引入了一个新类`ReflectiveOperationException`可以帮你捕获这些反射异常:

```java
try {
    Class<?> clazz = Class.forName("com.biezhi.apple.User");
    clazz.getMethods()[0].invoke(object);
} catch (ReflectiveOperationException e){
    e.printStackTrace();
}
```

## 文件操作

我们知道在JDK6甚至之前的时候，我们想要读取一个文本文件也是非常麻烦的一件事，而现在他们都变得简单了，
这要归功于`NIO2`，我们先看看之前的做饭:

**读取一个文本文件**

```java
BufferedReader br = null;
try {
    new BufferedReader(new FileReader("file.txt"));
    StringBuilder sb = new StringBuilder();
    String line      = br.readLine();
    while (line != null) {
        sb.append(line);
        sb.append(System.lineSeparator());
        line = br.readLine();
    }
    String everything = sb.toString();
} catch (Exception e){
    e.printStackTrace();
} finally {
    try {
        br.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

大家对这样的一段代码一定不陌生，但这样太繁琐了，我只想读取一个文本文件，要写这么多代码还要
处理让人头大的一堆异常，怪不得别人吐槽Java臃肿，是在线输了。。。

下面我要介绍在JDK7中是如何改善这些问题的。

### Path

`Path`用于来表示文件路径和文件，和`File`对象类似，`Path`对象并不一定要对应一个实际存在的文件，
它只是一个路径的抽象序列。

要创建一个`Path`对象有多种方法，首先是final类Paths的两个static方法，如何从一个路径字符串来构造Path对象：

```java
Path path1   = Paths.get("/home/biezhi", "a.txt");
Path path2   = Paths.get("/home/biezhi/a.txt");

URI  u       = URI.create("file:////home/biezhi/a.txt");
Path pathURI = Paths.get(u);
```

通过`FileSystems`构造

```java
Path filePath = FileSystems.getDefault().getPath("/home/biezhi", "a.txt");
```

`Path`、`URI`、`File`之间的转换

```java
File file  = new File("/home/biezhi/a.txt");
Path p1    = file.toPath();
p1.toFile();
file.toURI();
```

### 读写文件

你可以使用`Files`类快速实现文件操作，例如读取文件内容:

```java
byte[] data    = Files.readAllBytes(Paths.get("/home/biezhi/a.txt"));
String content = new String(data, StandardCharsets.UTF_8);
```

如果希望按照行读取文件，可以调用

```java
List<String> lines = Files.readAllLines(Paths.get("/home/biezhi/a.txt"));
```

反之你想将字符串写入到文件可以调用

```java
Files.write(Paths.get("/home/biezhi/b.txt"), "Hello JDK7!".getBytes());
```

你也可以按照行写入文件，`Files.write`方法的参数中支持传递一个实现`Iterable`接口的类实例。
将内容追加到指定文件可以使用`write`方法的第三个参数`OpenOption`:

```java
Files.write(Paths.get("/home/biezhi/b.txt"), "Hello JDK7!".getBytes(),
 StandardOpenOption.APPEND);
```

> 默认情况Files类中的所有方法都会使用UTF-8编码进行操作，当你不愿意这么干的时候可以传递Charset参数进去变更。

当前`Files`还有一些其他的常用方法:

```java
InputStream ins  = Files.newInputStream(path);
OutputStream ops = Files.newOutputStream(path);
Reader reader    = Files.newBufferedReader(path);
Writer writer    = Files.newBufferedWriter(path);
```

### 创建、移动、删除

创建文件、目录

```java
if (!Files.exists(path)) {
    Files.createFile(path);
    Files.createDirectory(path);
}
```

`Files`还提供了一些方法让我们创建临时文件/临时目录:

```java
Files.createTempFile(dir, prefix, suffix);
Files.createTempFile(prefix, suffix);
Files.createTempDirectory(dir, prefix);
Files.createTempDirectory(prefix);
```

这里的`dir`是一个`Path`对象，并且字符串prefix和suffix都可能为null。
例如调用`Files.createTempFile(null, ".txt")`会返回一个类似`/tmp/21238719283331124678.txt`

> 读取一个目录下的文件请使用`Files.list`和`Files.walk`方法

复制、移动一个文件内容到某个路径

```java
Files.copy(in, path);
Files.move(path, path);
```

删除一个文件

```java
Files.delete(path);
```

## 小的改进

Java8是一个较大改变的版本，包含了API和库方面的修正，它还对我们常用的API进行很多微小的调整，
下面我会带你了解字符串、集合、注解等新方法。

### 字符串

使用过JavaScript语言的人可能会知道当我们将一个数组中的元素组合起来变成字符串有一个方法`join`，
例如我们经常用到将数组中的字符串拼接成用逗号分隔的一长串，这在Java中是要写for循环来完成的。

Java8种添加了`join`方法帮你搞定这一切:

```java
String str = String.join(",", "a", "b", "c");
```

第一个参数是分隔符，后面接收一个`CharSequence`类型的可变参数数组或一个`Iterable`。

### 集合

集合改变中最大的当属前面章节中提到的`Stream API`，除此之外还有一些小的改动。

| 类/接口  | 新方法 |
| ------ | ---- |
| Iterable | foreach |
| Collection | removeIf |
| List | replaceAll, sort |
| Map | forEach, replace, replaceAll, remove(key, value),<br/> putIfAbsent, compute, computeIf, merge |
| Iterator | forEachRemaining |
| BitSet | stream |

- Map中的很多方法对并发访问十分重要，我们将在后面的章节中介绍
- Iterator提供forEachRemaining将剩余的元素传递给一个函数
- BitSet可以产生一个Stream对象

**通用目标类型判断**

Java8对泛型参数的推断进行了增强。相信你对Java8之前版本中的类型推断已经比较熟悉了。
比如，Java中的方法`emptyList`方法定义如下：

```java
static <T> List<T> emptyList();
```

`emptyList`方法使用了类型参数T进行参数化。
你可以像下面这样为该类型参数提供一个显式的类型进行函数调用：

```java
List<Person> persons = Collections.<Person>emptyList();
```

不过Java也可以推断泛型参数的类型，上面的代码和下面这段代码是等价的：

```java
List<Person> persons = Collections.emptyList();
```

我还是习惯于这样书写。

### 注解

Java 8在两个方面对注解机制进行了改进，分别为：

- 可以定义重复注解
- 可以为任何类型添加注解

### 重复注解

之前版本的Java禁止对同样的注解类型声明多次。由于这个原因，下面的第二句代码是无效的：

```java
@interface Basic {
    String name();
}

@Basic(name="fix")
@Basic(name="todo")
class Person{ }
```

我们之前可能会通过数组的做法绕过这一限制:

```java
@interface Basic {
    String name();
}

@interface Basics {
    Basic[] value();
}

@Basics( { @Basic(name="fix") , @Basic(name="todo") } )
class Person{ }
```

Book类的嵌套注解相当难看。这就是Java8想要从根本上移除这一限制的原因，去掉这一限制后，
代码的可读性会好很多。现在，如果你的配置允许重复注解，你可以毫无顾虑地一次声明多个同一种类型的注解。
它目前还不是默认行为，你需要显式地要求进行重复注解。

**创建一个重复注解**

如果一个注解在设计之初就是可重复的，你可以直接使用它。但是，如果你提供的注解是为用户提供的，
那么就需要做一些工作，说明该注解可以重复。下面是你需要执行的两个步骤：

1. 将注解标记为`@Repeatable`
2. 提供一个注解的容器下面的例子展示了如何将`@Basic`注解修改为可重复注解

```java
@Repeatable(Basics.class)
@interface Basic {
    String name();
}

@Retention(RetentionPolicy.RUNTIME)
@interface Basics {
    Basic[] value();
}
```

完成了这样的定义之后，`Person`类可以通过多个`@Basic`注解进行注释，如下所示：

```java
@Basic(name="fix")
@Basic(name="todo")
class Person{ }
```

编译时， Person 会被认为使用了 `@Basics( { @Basic(name="fix") , @Basic(name="todo")} )`
这样的形式进行了注解，所以，你可以把这种新的机制看成是一种语法糖，
它提供了程序员之前利用的惯用法类似的功能。为了确保与反射方法在行为上的一致性，
注解会被封装到一个容器中。 Java API中的`getAnnotation(Class<T> annotationClass)`方法会为注解元素返回类型为`T`的注解。
如果实际情况有多个类型为`T`的注解，该方法的返回到底是哪一个呢？

我们不希望一下子就陷入细节的魔咒，类`Class`提供了一个新的`getAnnotationsByType`方法，
它可以帮助我们更好地使用重复注解。比如，你可以像下面这样打印输出`Person`类的所有`Basic`注解：

返回一个由重复注解`Basic`组成的数组

```java
public static void main(String[] args) {
    Basic[] basics = Person.class.getAnnotationsByType(Basic.class);
    Arrays.asList(basics).forEach(a -> { System.out.println(a.name()); }); }
```

### Null检查

`Objects`类添加了两个静态方法`isNull`和`nonNull`，在使用流的时候非常有用。

例如获取一个流的所有不为null的对象:

```java
Stream.of("a", "c", null, "d")
        .filter(Objects::nonNull)
        .forEach(System.out::println);
```

## Optional

空指针异常一直是困扰Java程序员的问题，也是我们必须要考虑的。当业务代码中充满了`if else`判断`null`
的时候程序变得不再优雅，在Java8中提供了`Optional`类为我们解决`NullPointerException`。

我们先来看看这段代码有什么问题?

```java
class User {
    String name;
    public String getName() {
        return name;
    }
}

public static String getUserName(User user){
    return user.getName();
}
```

这段代码看起来很正常，每个`User`都会有一个名字。所以调用`getUserName`方法会发生什么呢？
实际这是不健壮的程序代码，当`User`对象为null的时候会抛出一个空指针异常。

我们普遍的做法是通过判断`user != null`然后获取名称

```java
public static String getUserName(User user){
    if(user != null){
        return user.getName();
    }
    return null;
}
```

但是如果对象嵌套的层次比较深的时候这样的判断我们需要编写多少次呢？难以想象

### 处理空指针

**使用Optional优化代码**

```java
public static String getUserNameByOptional(User user) {
    Optional<String> userName = Optional.ofNullable(user).map(User::getName);
    return userName.orElse(null);
}
```

当`user`为null的时候我们设置UserName的值为null，否则返回`getName`的返回值，但此时不会抛出空指针。

在之前的代码片段中是我们最熟悉的命令式编程思维，写下的代码可以描述程序的执行逻辑，得到什么样的结果。
后面的这种方式是函数式思维方式，在函数式的思维方式里，结果比过程更重要，不需要关注执行的细节。程序的具体执行由编译器来决定。
这种情况下提高程序的性能是一个不容易的事情。

我们再次了解下`Optional`中的一些使用方法

### Optional方法

**创建 Optional 对象**

你可以通过静态工厂方法`Optional.empty`，创建一个空的Optional对象：

```java
Optional<User> emptyUser = Optional.empty();
```

创建一个非空值的Optional

```java
Optional<User> userOptional = Optional.of(user);
```

如果user是一个null，这段代码会立即抛出一个`NullPointerException`，而不是等到你试图访问`user`的属性值时才返回一个错误。

可接受null的Optional

```java
Optional<User> ofNullOptional = Optional.ofNullable(user);
```

使用静态工厂方法`Optional.ofNullable`，你可以创建一个允许null值的Optional对象。

如果user是null，那么得到的Optional对象就是个空对象，但不会让你导致空指针。

使用`map`从Optional对象中提取和转换值

```java
Optional<User> ofNullOptional = Optional.ofNullable(user);
Optional<String> userName = ofNullOptional.map(User::getName);
```

这种操作就像我们之前在操作`Stream`是一样的，获取的只是`User`中的一个属性。

**默认行为及解引用Optional对象**

我们决定采用orElse方法读取这个变量的值，使用这种方式你还可以定义一个默认值，
遭遇空的Optional变量时，默认值会作为该方法的调用返回值。
Optional类提供了多种方法读取 Optional实例中的变量值。

- get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量 值，否则就抛出一个NoSuchElementException异常。所以，除非你非常确定Optional 变量一定包含值，否则使用这个方法是个相当糟糕的主意。此外，这种方式即便相对于 嵌套式的null检查，也并未体现出多大的改进。
- orElse(T other)是我们在代码清单10-5中使用的方法，正如之前提到的，它允许你在 Optional对象不包含值时提供一个默认值。
- orElseGet(Supplier<? extends T> other)是orElse方法的延迟调用版，Supplier 方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作， 你应该考虑采用这种方式（借此提升程序的性能），或者你需要非常确定某个方法仅在 Optional为空时才进行调用，也可以考虑该方式（这种情况有严格的限制条件）。
- orElseThrow(Supplier<? extends X> exceptionSupplier)和get方法非常类似， 它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希 望抛出的异常类型。
- ifPresent(Consumer<? super T>)让你能在变量值存在时执行一个作为参数传入的 方法，否则就不进行任何操作。

当前除了这些Optional类也具备一些和Stream类似的API，我们先看看Optional类方法:

| 方法  | 描述 |
| ------ | ---- |
| empty | 返回一个空的 Optional 实例 |
| get | 如果该值存在，将该值用Optional包装返回，否则抛出一个`NoSuchElementException`异常 |
| ifPresent | 如果值存在，就执行使用该值的方法调用，否则什么也不做 |
| isPresent | 如果值存在就返回true，否则返回false |
| filter | 如果值存在并且满足提供的谓词，就返回包含该值的 Optional 对象;<br/>否则返回一个空的Optional对象 |
| map | 如果值存在，就对该值执行提供的 mapping 函数调用 |
| flatMap | 如果值存在，就对该值执行提供的 mapping 函数调用，<br/>返回一个 Optional 类型的值，否则就返 回一个空的Optional对象 |
| of | 将指定值用 Optional 封装之后返回，如果该值为null，则抛出一个`NullPointerException`异常 |
| ofNullable | 将指定值用 Optional 封装之后返回，如果该值为 null，则返回一个空的Optional对象 |
| orElse | 如果有值则将其返回，否则返回一个默认值 |
| orElseGet | 如果有值则将其返回，否则返回一个由指定的 Supplier 接口生成的值 |
| orElseThrow | 如果有值则将其返回，否则抛出一个由指定的 Supplier 接口生成的异常 |

**用Optional封装可能为null的值**

目前我们写的大部分Java代码都会使用返回NULL的方式来表示不存在值，比如Map中通过Key获取值，
当不存在该值会返回一个null。
但是，正如我们之前介绍的，大多数情况下，你可能希望这些方法能返回一个Optional对象。
你无法修改这些方法的签名，但是你很容易用Optional对这些方法的返回值进行封装。

我们接着用Map做例子，假设你有一个`Map<String, Object>`类型的map，访问由key的值时，
如果map中没有与key关联的值，该次调用就会返回一个null。

```java
Object value = map.get("key");
```

使用Optional封装map的返回值，你可以对这段代码进行优化。要达到这个目的有两种方式：
你可以使用笨拙的`if-then-else`判断语句，毫无疑问这种方式会增加代码的复杂度；
或者你可以采用`Optional.ofNullable`方法

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

每次你希望安全地对潜在为null的对象进行转换，将其替换为Optional对象时，都可以考虑使用这种方法。

参考资料：[Java文件IO操作应该抛弃File拥抱Paths和Files](http://www.cnblogs.com/digdeep/p/4478734.html)