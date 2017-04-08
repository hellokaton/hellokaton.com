---
layout: post
title: 四种方法使Map线程安全
categories: java
tags: hashmap 线程安全
---

如果需要使 Map 线程安全，大致有这么四种方法： 

## 1、使用 synchronized 关键字，这也是最原始的方法。代码如下 

```java
synchronized(anObject) {   
     value = map.get(key);   
}
```

JDK1.2 提供了 `Collections.synchronizedMap(originMap)` 方法，同步方式其实和上面这段代码相同。 

<!-- more -->

## 2、使用 JDK1.5 提供的锁（java.util.concurrent.locks.Lock）。代码如下 

```java
lock.lock();   
value = map.get(key);   
lock.unlock(); 
```

## 3、实际应用中，可能多数操作都是读操作，写操作较少。

针对这种情况，可以使用 JDK1.5 提供的读写锁（java.util.concurrent.locks.ReadWriteLock）。代码如下 

```java
rwlock.readLock().lock();   
value = map.get(key);   
rwlock.readLock().unlock(); 
``` 

这样两个读操作可以同时进行，理论上效率会比方法 2 高。 

## 4、使用 JDK1.5 提供的 java.util.concurrent.ConcurrentHashMap 类。

该类将 Map 的存储空间分为若干块，每块拥有自己的锁，大大减少了多个线程争夺同一个锁的情况。代码如下 

```java
value = map.get(key); //同步机制内置在 get 方法中  
```

写了段测试代码，针对这四种方式进行测试，结果见附图。

测试内容为 1 秒钟所有 get 方法调用次数的总和。为了比较，增加了未使用任何同步机制的情况（非安全！）。理论上，不同步应该最快。 

更多线程时，CPU 利用率提高，但增加了线程调度的开销，测试结果与五个线程差不多。 

从附图可以看出： 

1. 不同步确实最快，与预期一致。 
2. 四种同步方式中，ConcurrentHashMap 是最快的，接近不同步的情况。 
3. synchronized 关键字非常慢，比使用锁慢了两个数量级。真是大跌眼镜，我很迷惑为什会 synchronized 慢到这个程度。 
4. 使用读写锁的读锁，比普通所稍慢。这个比较意外，可能硬件或测试代码没有发挥出读锁的全部功效。 

## 结论： 

1. 如果 `ConcurrentHashMap` 够用，则使用 `ConcurrentHashMap`。 
2. 如果需自己实现同步，则使用 JDK1.5 提供的锁机制，避免使用 `synchronized` 关键字。