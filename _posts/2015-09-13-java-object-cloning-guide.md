---
layout: post
title: Java对象克隆指南
categories: java
tags: clone
---

在java中，克隆是一个精确的原始拷贝，这基本上意味着能够创建一个对象相似的状态与原始对象。clone()方法提供了这种功能。在这篇文章中,我们将探讨java克隆的最重要的方面。

 - 详细解释了克隆
 - Java基础克隆
 - 浅克隆
 - 深克隆
 - 复制构造函数
 - 序列化克隆
 - 使用Apache commons克隆
 - 最佳实践

<!-- more -->

## 详细解释了克隆

克隆是关于创建原始对象的副本，其词典意义是:“使一个完全相同的副本”。
**默认情况下,java克隆“字段复制”，即是对象类没有了解的结构类的clone()方法将被调用**。所以，当调用JVM的克隆，做以下的事情：

1）如果该类**只有原始数据类型成员**，则将创建一个完全新的对象副本，并将**返回新的对象副本的引用**。

2）如果该类包含**任何类类型的成员**，则复制该对象引用的对象引用，因此在**原对象和克隆对象引用相同的对象**时，该成员引用。

除了上面的默认行为，您可以随时重写此行为并指定您自己的。这是通过使用重写clone()方法。让我们看看如何做。

## Java基础克隆

每一种支持克隆对象的语言都有它自己的规则，所以它也有其自身的规则。在Java里，如果一个类需要支持克隆它必须做以下的事情：

A）你必须实现Cloneable接口。

B）你必须重写Object类的clone方法。

Java文档中中关于Clone方法的解释如下：

```java
/*
Creates and returns a copy of this object. The precise meaning of "copy" may depend on the class of the object.
The general intent is that, for any object x, the expression:
1) x.clone() != x will be true
2) x.clone().getClass() == x.getClass() will be true, but these are not absolute requirements.
3) x.clone().equals(x) will be true, this is not an absolute requirement.
*/
protected native Object  [More ...] clone() throws CloneNotSupportedException;
```

1. 第一次声明保证克隆对象将有单独的内存地址分配。
2. 二次声明表明，原始和克隆的对象应该具有相同的类类型，但它不是强制性的。
3. 第三声明表明，原始和克隆的对象应该是平等的equals()方法使用，但它不是强制性的。

我们在实际代码片段中使用Clone：

我们的第一个例子是3个属性的Employee类，身份证，姓名和部门。

```java
public class Employee implements Cloneable{
 
    private int empoyeeId;
    private String employeeName;
    private Department department;
 
    public Employee(int id, String name, Department dept) {
        this.empoyeeId = id;
        this.employeeName = name;
        this.department = dept;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

部门有2个属性，id和名称

```java
public class Department {
    private int id;
    private String name;
 
    public Department(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

所以，如果我们需要克隆员工类，那么我们需要做的事情如下：

```java
public class TestCloning {
 
    public static void main(String[] args) throws CloneNotSupportedException {
        Department dept = new Department(1, "Human Resource");
        Employee original = new Employee(1, "Admin", dept);
        //创建一个原始对象的克隆
        Employee cloned = (Employee) original.clone();
        //验证克隆后的员工id
        System.out.println(cloned.getEmpoyeeId());
 
        //JDK的验证规则
 
        //输出true,对象必须有不同的内存地址
        System.out.println(original != cloned);
 
        //正如我们正在返回同一个类，所以它应该是true
        System.out.println(original.getClass() == cloned.getClass());
 
        //如果我们想让它为true，在Employee类上我们需要重写equals方法。
        System.out.println(original.equals(cloned));
    }
}
输出:
1
true
true
false
```

很好，我们成功克隆了员工对象。但是记住，我们有一个相同的对象引用，现在不管改变哪个都在修改对象的状态。想想该怎么解决？
来看这里：

```java
public class TestCloning {
 
    public static void main(String[] args) throws CloneNotSupportedException {
        Department hr = new Department(1, "Human Resource");
        Employee original = new Employee(1, "Admin", hr);
        Employee cloned = (Employee) original.clone();
 
        //让我们在克隆的对象中修改部门名称，在原对象中验证
        cloned.getDepartment().setName("Finance");
     
        System.out.println(original.getDepartment().getName());
    }
}
输出: Finance
```

哎呀，克隆对象的更改也是可见的。这样，克隆的对象可以使系统的破坏，如果允许这样做，任何人都可以来克隆你的应用对象，做他喜欢做的任何事。我们能阻止这个吗？？
答案是肯定的，我们可以。我们可以防止这种使用深度克隆和使用复制构造函数。我们将在这篇文章后面了解他们。

## 浅克隆

这是在java中的默认实现。在重写的克隆方法时，如果你不是克隆的所有对象类型（而不是primitives），那么是一个浅拷贝。
以上所有的例子都是浅拷贝的，因为我们没有克隆到员工类clone()方法。现在，我将继续到下一节，我们将看到深克隆。

## 深克隆

在大多数情况下，这是预期的行为。我们想要进行一个克隆，它是独立的，并在克隆的时候进行的修改不应该影响原来的。
让我们看看如何在我们如下的案例中完成它。

```java
//在员工类中修改clone方法
@Override
protected Object clone() throws CloneNotSupportedException {
    Employee cloned = (Employee)super.clone();
    cloned.setDepartment((Department)cloned.getDepartment().clone());
    return cloned;
}
```

我修改了员工类类的clone()方法和添加以下部门类的clone()方法。

```java
// 在部门类中定义clone()方法
@Override
protected Object clone() throws CloneNotSupportedException {
    return super.clone();
}
```

现在测试我们的克隆代码给出了预期的结果，并不会修改部门名称。

```java
public class TestCloning {
    public static void main(String[] args) throws CloneNotSupportedException {
        Department hr = new Department(1, "Human Resource");
        Employee original = new Employee(1, "Admin", hr);
        Employee cloned = (Employee) original.clone();
 
        //让我们在克隆的对象中更改部门名称，在原对象中验证
        cloned.getDepartment().setName("Finance");
 
        System.out.println(original.getDepartment().getName());
    }
}
输出:
Human Resource
```

在这里，克隆的对象的状态变化不影响原来的对象。
因此深度克隆需要满足以下规则。

- 无需单独复制primitives
- 原始类的所有类成员应该支持克隆,克隆方法的原始类的上下文应该叫super.clone()。
- 如果任何成员类不支持克隆，然后在克隆方法中，就必须创建一个新的成员类的实例，并将其所有属性复制到新的成员类对象中。这个新的成员类对象将被设置在克隆对象中。

## 复制构造函数

拷贝构造函数是特殊类中的构造函数将参数的类类型。所以,当你通过类的实例拷贝构造函数,构造函数将返回一个新实例的类的值复制参数实例。让我们看看这个例子:

```java
public class PointOne {
    private Integer x;
    private Integer y;
 
    public PointOne(PointOne point){
        this.x = point.x;
        this.y = point.y;
    }
}
```

这个方法看起来很简单。当你定义一个类的时候，你需要定义一个类似的构造函数。在子类中，你需要复制子类的属性，然后把参数传递给超级类的构造函数。让我们看看怎样？

```java
public class PointTwo extends PointOne{
    private Integer z;
 
    public PointTwo(PointTwo point){
        super(point); //调用超类构造函数
        this.z = point.z;
    }
}
```

那么,现在这样好吗?不好，继承的问题是具体行为只在运行时确定。所以,在我们的例子中,如果一些类通过PointOne PointTwo的实例构造函数。在这种情况下,你会得到PointOne的实例返回PointTwo实例作为参数传递。让我们看看这个代码:

```java
class Test {
    public static void main(String[] args)
    {
        PointOne one = new PointOne(1,2);
        PointTwo two = new PointTwo(1,2,3);
 
        PointOne clone1 = new PointOne(one);
        PointOne clone2 = new PointOne(two);
 
        //检查类类型
        System.out.println(clone1.getClass());
        System.out.println(clone2.getClass());
    }
}
输出:
class corejava.cloning.PointOne
class corejava.cloning.PointOne
```

**创建一个拷贝构造函数的另一种方法是静态工厂方法**。他们上课输入参数,使用另一个类的构造函数创建一个新的实例。然后这些工厂方法将所有状态的数据复制到上一步中创建的新类实例,并返回这个更新实例。

```java
public class PointOne {
    private Integer x;
    private Integer y;
 
    public PointOne(Integer x, Integer y) {
        this.x = x;
        this.y = y;
    }
 
    public PointOne copyPoint(PointOne point) throws CloneNotSupportedException {
        if(!(point instanceof Cloneable)) {
            throw new CloneNotSupportedException("Invalid cloning");
        }
        //做更多其他的事
        return new PointOne(point.x, point.y);
    }
}
```

## 序列化克隆

这是另一种简单方式的深层克隆。在这种方法中，你只需序列化和反序列化被克隆对象。显然，这需要被克隆的对象应实现Serializable接口。
在进行任何进一步的行动之前，我要告诫你，这种技术是不可使用的。首先，序列化是非常昂贵的。它很可能是一百倍的clone方法。其次，不是所有的对象都是可序列化的。再次，使类可序列化是很困难的，不是所有的类可以指望得到它的权利。

让我们看看代码示例：

```java
@SuppressWarnings("unchecked")
public static  T clone(T t) throws Exception {
    //如果T是可序列化则克隆，否则抛出CloneNotSupportedException异常
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
 
    //序列化操作
    serializeToOutputStream(t, bos);
    byte[] bytes = bos.toByteArray();
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bytes));
 
    //反序列化并返回新实例
    return (T)ois.readObject();
}
```


## 使用Apache commons克隆

[Apache commons](http://commons.apache.org/proper/commons-lang)也提供深克隆的函数。如果你觉得感兴趣的下他们的官方文档。下面是样品的使用克隆工具使用Apache commons:

```java
SomeObject cloned = org.apache.commons.lang.SerializationUtils.clone(someObject);
```

## 最佳实践

1) 当你不知道你是否可以调用clone()方法的类你不确定如果是在这个类中实现,您可以检查和类是否实现`Cloneable`接口如下。

```java
if(obj1 instanceof Cloneable){
    obj2 = obj1.clone();
}

//不要这样做，Cloneabe中没有任何方法
obj2 = (Cloneable)obj1.clone();
```

2）没有构造函数被称为被克隆的对象。因此，这是你的责任，以确保所有的成员已被正确设置。此外，如果你在系统中跟踪对象的数目，通过计算构造函数的调用，你得到一个新的额外递增计数器。