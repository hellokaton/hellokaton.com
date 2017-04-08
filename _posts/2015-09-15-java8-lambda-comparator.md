---
layout: post
title: Java8 Lambda表达式之比较器
categories: java
tags: java8 lambda comparator
---

在这个例子中，我将向你展示如何使用Java8的lambda表达式写的比较器排序列表。

1.经典`Comparator`例子

```java
Comparator<Developer> byName = new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getName().compareTo(o2.getName());
	}
};
```

<!-- more -->

2.Lambda表达式方式

```java
Comparator<Developer> byName = 
		(Developer o1, Developer o2)->o1.getName().compareTo(o2.getName());
```

### 1. 使用Lambda排序

这个例子使用年龄比较`Developer`对象，通常你使用`Collections.sort`并且通过一个匿名函数实现`Comparator`：

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class TestSorting {

	public static void main(String[] args) {

		List<Developer> listDevs = getDevelopers();

		System.out.println("Before Sort");
		for (Developer developer : listDevs) {
			System.out.println(developer);
		}
		
		//sort by age
		Collections.sort(listDevs, new Comparator<Developer>() {
			@Override
			public int compare(Developer o1, Developer o2) {
				return o1.getAge() - o2.getAge();
			}
		});
	
		System.out.println("After Sort");
		for (Developer developer : listDevs) {
			System.out.println(developer);
		}
		
	}

	private static List<Developer> getDevelopers() {

		List<Developer> result = new ArrayList<Developer>();

		result.add(new Developer("mkyong", new BigDecimal("70000"), 33));
		result.add(new Developer("alvin", new BigDecimal("80000"), 20));
		result.add(new Developer("jason", new BigDecimal("100000"), 10));
		result.add(new Developer("iris", new BigDecimal("170000"), 55));
		
		return result;

	}
	
}
```

输出

```bash
Before Sort
Developer [name=mkyong, salary=70000, age=33]
Developer [name=alvin, salary=80000, age=20]
Developer [name=jason, salary=100000, age=10]
Developer [name=iris, salary=170000, age=55]

After Sort
Developer [name=jason, salary=100000, age=10]
Developer [name=alvin, salary=80000, age=20]
Developer [name=mkyong, salary=70000, age=33]
Developer [name=iris, salary=170000, age=55]
```

当排序的需求要更改时，你需要通过一个新的`Comparator`比较器：

```java
//sort by age
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getAge() - o2.getAge();
	}
});

//sort by name	
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getName().compareTo(o2.getName());
	}
});
			
//sort by salary
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getSalary().compareTo(o2.getSalary());
	}
});		
```

上面的代码可以工作，但是，你认为创建一个类只是因为你想改变一个单一的代码，不觉得有点奇怪吗？

### 2. 使用Lambda排序

在Java8中，`List`支持`sort`方法，不需要使用`Collections.sort`了

```java
//List.sort() since Java 8
listDevs.sort(new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o2.getAge() - o1.getAge();
	}
});	
```

Lambda表达式示例：

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class TestSorting {

	public static void main(String[] args) {

		List<Developer> listDevs = getDevelopers();
		
		System.out.println("Before Sort");
		for (Developer developer : listDevs) {
			System.out.println(developer);
		}
		
		System.out.println("After Sort");
		
		//lambda here!
		listDevs.sort((Developer o1, Developer o2)->o1.getAge()-o2.getAge());
	
		//java 8 only, lambda also, to print the List
		listDevs.forEach((developer)->System.out.println(developer));
	}

	private static List<Developer> getDevelopers() {

		List<Developer> result = new ArrayList<Developer>();

		result.add(new Developer("mkyong", new BigDecimal("70000"), 33));
		result.add(new Developer("alvin", new BigDecimal("80000"), 20));
		result.add(new Developer("jason", new BigDecimal("100000"), 10));
		result.add(new Developer("iris", new BigDecimal("170000"), 55));
		
		return result;

	}
	
}
```

输出

```bash
Before Sort
Developer [name=mkyong, salary=70000, age=33]
Developer [name=alvin, salary=80000, age=20]
Developer [name=jason, salary=100000, age=10]
Developer [name=iris, salary=170000, age=55]

After Sort
Developer [name=jason, salary=100000, age=10]
Developer [name=alvin, salary=80000, age=20]
Developer [name=mkyong, salary=70000, age=33]
Developer [name=iris, salary=170000, age=55]
```

### 3. 更多Lambda表达式例子

3.1 根据年龄排序

```java
//sort by age
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getAge() - o2.getAge();
	}
});

//lambda
listDevs.sort((Developer o1, Developer o2)->o1.getAge()-o2.getAge());

//lambda, valid, parameter type is optional
listDevs.sort((o1, o2)->o1.getAge()-o2.getAge());
```

3.2 根据姓名排序

```java
//sort by name
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getName().compareTo(o2.getName());
	}
});
	
//lambda
listDevs.sort((Developer o1, Developer o2)->o1.getName().compareTo(o2.getName()));		

//lambda
listDevs.sort((o1, o2)->o1.getName().compareTo(o2.getName()));
```

3.3 根据薪水排序

```java
//sort by salary
Collections.sort(listDevs, new Comparator<Developer>() {
	@Override
	public int compare(Developer o1, Developer o2) {
		return o1.getSalary().compareTo(o2.getSalary());
	}
});				

//lambda
listDevs.sort((Developer o1, Developer o2)->o1.getSalary().compareTo(o2.getSalary()));

//lambda
listDevs.sort((o1, o2)->o1.getSalary().compareTo(o2.getSalary()));
```

3.4 排序反转

3.4.1 使用lambda表达式对薪水进行排序

```java
Comparator<Developer> salaryComparator = (o1, o2)->o1.getSalary().compareTo(o2.getSalary());
listDevs.sort(salaryComparator);
```

输出

```bash
Developer [name=mkyong, salary=70000, age=33]
Developer [name=alvin, salary=80000, age=20]
Developer [name=jason, salary=100000, age=10]
Developer [name=iris, salary=170000, age=55]
```

3.4.2 使用lambda表达式对薪水进行反转排序

```java
Comparator<Developer> salaryComparator = (o1, o2)->o1.getSalary().compareTo(o2.getSalary());
listDevs.sort(salaryComparator.reversed());
```

输出：

```bash
Developer [name=iris, salary=170000, age=55]
Developer [name=jason, salary=100000, age=10]
Developer [name=alvin, salary=80000, age=20]
Developer [name=mkyong, salary=70000, age=33]
```

参考资料：

1. [Start Using Java Lambda Expressions](http://www.developer.com/java/start-using-java-lambda-expressions.html)
2. [Oracle : Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
3. [Oracle : Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)