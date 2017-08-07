---
title: 跟上Java8 - 使用Java8进行并发编程
date: 2017-07-22
category: ["跟上Java8"]
tags: ["Java8", "concurrent", "并发"]
---

并发编程长期以来都是考验程序员的难题，多核使得程序处理性能得到提升。
同时也对编程人员的能力要求更高，多线程、并发的知识点一语难详，
在本章节中我们一起学习一下Java8种的计数器、并发Map、StampedLock等知识。

## 计数器

我们经常会因为多线程的操作而头疼，在 `java.concurrent.atomic` 包下有很多
类帮助我们进行一些原子操作，并且是线程安全的，当然最笨的方法是用 `synchronized`
关键字去加锁处理。这些原子类的内部使用了 [CAS](http://en.wikipedia.org/wiki/Compare-and-swap)
这种操作，大多数CPU都支持原子指令。通常这种方式比使用锁的性能要更好，所以我更喜欢用
原子类去实现修改变量的操作。

> 这个包里面提供了一组原子变量类。其基本的特性就是在多线程环境下，
> 当有多个线程同时执行这些类的实例包含的方法时，具有排他性，
> 即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，
> 才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。
> 实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。
> 可以对基本数据、数组中的基本数据、对类中的基本数据进行操作。
> 原子变量类相当于一种泛化的volatile变量，能够支持原子的和有条件的读-改-写操作。

下面我们举几个例子看看在JDK1.5中提供的 `AtomicInteger` 是如何使用的。

### AtomicInteger

**10个线程对数据进行递增操作**

```java
AtomicInteger   atomicInt = new AtomicInteger(0);
ExecutorService executor  = Executors.newFixedThreadPool(10);

IntStream.range(0, 500)
        .forEach(i -> executor.submit(() -> {
            String msg = String.format("当前线程: %s => %d",
                Thread.currentThread().getName(), atomicInt.incrementAndGet());
            System.out.println(msg);
        }));

stop(executor);

System.out.println(atomicInt.get()); // 500
```

线程池的 `stop` 方法

```java
public static void stop(ExecutorService executor) {
    try {
        executor.shutdown();
        executor.awaitTermination(60, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        System.err.println("termination interrupted");
    } finally {
        if (!executor.isTerminated()) {
            System.err.println("killing non-finished tasks");
        }
        executor.shutdownNow();
    }
}
```

如果你运行上面的代码会多个线程交替输出，但是不会出现重复累加的操作。
最终会输出500，我们调用 `AtomicInteger` 的 `incrementAndGet` 方法进行递增并
返回递增后的值，这是一个原子操作，可以保证线程安全。

在1.8的API中为它添加了一个新方法 `updateAndGet`，接收一个类型为`IntUnaryOperator`
的lambda表达式，方便你做一些运算操作

```java
AtomicInteger   atomicInt = new AtomicInteger(0);
ExecutorService executor  = Executors.newFixedThreadPool(10);

IntStream.range(0, 500)
        .forEach(i -> executor.submit(() -> atomicInt.updateAndGet(n -> n + 2)));

stop(executor);

System.out.println(atomicInt.get());    // => 1000
```

我们使用10个线程对该数据进行500次循环，每次将当前数字进行`+2`，最终得到的结果为1000。

除此之外，还有一些其他的类你也可以尝试的类，原子操作类的atomic包，里面包含了

1. 布尔类型的AtomicBoolean
2. 整型AtomicInteger、AtomicIntegerArray、AtomicIntegerFieldUpdater
3. 长整型AtomicLong、AtomicLongArray、AtomicLongFieldUpdater
4. 引用型AtomicMarkableReference、AtomicReference、AtomicReferenceArray、AtomicReferenceFieldUpdater、AtomicStampedReference
5. 累加器DoubleAccumulator、DoubleAdder、LongAccumulator、LongAdder、Striped64

### LongAdder

`LongAdder`作为替代`AtomicLong`的类，可以用于连续地向数字添加值。

```java
LongAdder longAdder = new LongAdder();
ExecutorService executor  = Executors.newFixedThreadPool(10);

IntStream.range(0, 500)
        .forEach(i -> executor.submit(longAdder::increment));

stop(executor);

System.out.println(longAdder.sumThenReset());
```

`LongAdder`提供了方法`add()`，`increment()`，就像原子序数类一样，也是线程安全的。
但是，除了总结单个结果之外，这个类在内部维护一组变量以减少对线程的争用。
实际结果可以通过调用`sum()`方法或`sumThenReset()`。

当多线程的更新比读取操作更多的时候，该类的性能优于原子类。
比如你想得知道某个URL被访问了多少次，就可以用这个类。
`LongAdder`内部是一组变量被保存在内存中，缺点是内存消耗更高。

## ConcurrentMap

HashMap不是线程安全的Map，具体的原因我们不在这里累述，你可以看看[这篇文章](http://www.importnew.com/22011.html)进行了解。
那么我们如何保证线程安全的使用HashMap呢？有这么几种方式：

- Hashtable
- ConcurrentHashMap
- Synchronized Map

```java
//Hashtable
Map<String, String> hashtable = new Hashtable<>();

//synchronizedMap
Map<String, String> synchronizedHashMap = Collections.synchronizedMap(new HashMap<>());

//ConcurrentHashMap
Map<String, String> concurrentHashMap = new ConcurrentHashMap<>();
```

我们来看看ConcurrentHashMap，其他两个不是本节涉及的重点。

ConcurrentHashMap是线性安全的,多个线程不需要对内部结构造成破坏，就可以删除或者添加元素。
性能高，允许多个线程并发更新哈希表的不停部分，而不会造成相互堵塞。

```java
ConcurrentMap<String, String> map = new ConcurrentHashMap<>();
map.put("jack", "2016");
map.put("rose", "2017");
map.put("biezhi", "2018");

map.forEach((key, value) -> System.out.printf("%s = %s\n", key, value));
```

当我们需要对map中的值进行修改时以前的做法是

```java
String val = map.get("biezhi");
val = val == null ? "default" : val + " good!";
map.put("biezhi", val);
```

`ConcurrentHashMap`在更新数值的时候虽然是线程安全的，
但是在计算更新值的时候由于不能保证线程安全，更新的值可能是错误的。

在Java8中提供了`replace`方法，下面例子中当返回值不为true的时候可以再次更新。

```java
do {
    val = map.get("biezhi");
    newVal = val == null ? "default" : val + " good!";
} while (!map.replace("biezhi", val, newVal));
```

如果你的Value是数值型的，可以使用`LongAdder`类型或者`AtomicLong`来解决。
如果需要复杂计算，`compute`方法可以通过一个函数来计算新的值

```java
String word   = "biezhi";
String result = map.compute(word, (k, v) -> v == null ? "" : "找到了:" + v);
System.out.println(result);
```

`IfPresent`和`IfAbsent`方法分别表示已经存在值或者尚未存在值的情况下才进行操作。

`merge`方法可以在`key`第一次加入时做一些特殊操作，第二个参数表示键尚未存在时的初始值

```java
map.merge(word, "haha~", (existingValue, newValue) -> existingValue + newValue);
```

### 批量数据操作

java8为并发哈希映射提供了批量操作数据操作，即使在其他线程同时操作映射时也可以安全的执行。
批量数据操作会遍历映射并对匹配的元素进行操作。在批量操作过程中，不需要冻结映射的一个快照。
除非你恰好知道在这段时间内映射没有被修改，否则你应该将结果看作是映射状态的一个近似值。
批量操作有三类:

- `search`会对每个键值对提供一个lambda函数，直到函数返回非null将会终止并返回结果
- `reduce`提供一个lambda函数将所有键值对聚合起来
- `foreach`可以循环遍历map中的key，value

每个操作都有4个版本：

- operation Keys：对键操作
- operation Values：对值操作
- operation：对键和值操作
- operation Entries：对`Map.Entry`对象操作.

以search为例，有以下几个方法：

```java
U searchKeys(long threshold, BiFunction<? super K, ? extends U> f)
U searchValues(long threshold, BiFunction<? super V, ? extends U> f)
U search(long threshold, BiFunction<? super K, ? super V,? extends U> f)
U searchEntries(long threshold, BiFunction<Map.Entry<K, V>, ? extends U> f）
```

`threshold`为并行阀值，如果包含的元素数量超过阀值，操作会以并行方式执行，
如果希望永远以单线程执行，请使用`Long.MAX_VALUE`

`foreach`和`reduce`方法除了上述形式外，还有另一种形式，可以提供一个转换器函数，
首先会应用转换器函数，然后再将结果传递给消费者函数

```java
map.forEach(threshold,
(k, v) -> k + " -> " + v, // Transformer
System.out::println); // Consumer
```

这里`threshold`为设置的并行阀值，就像并行流那样，这些方法使用特定的`ForkJoinPool`，
由Java8中的`ForkJoinPool.commonPool()`提供。该池使用了取决于可用核心数量的预置并行机制。

对于int、long和double，reduce操作提供了专门的方法。
以toXXX开头，需要将输入值转换为原始类型值，并指定一个默认值和累加器函数

```java
long sum = map.reduceValuesToLong(threshold,
Long::longValue, // Transformer to primitive type
0, // Default value for empty map
Long::sum); // Primitive type accumulator
```

reduce操作将其输入与一个累加函数结合起来。下面的例子将所有的value用->连接起来

```java
String values = map.reduceValues(1, (value1, value2) -> value1 + "->" + value2);
System.out.println(values);
```

## StampedLock

Java8种提供了一种新锁 `StampedLock`，它和读写锁不同的是使用该锁或会返回一个long类型的标记，
你可以使用这些标记来释放锁，或者检查锁是否有效。除此之外，`StampedLock`支持乐观锁的模式。

我们写一个例子来看看：

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
StampedLock lock = new StampedLock();
Map<String, String> map = new HashMap<>();

executor.submit(() -> {
    long stamp = lock.writeLock();
    try {
        sleep(1);
        map.put("foo", "bar");
    } finally {
        lock.unlockWrite(stamp);
    }
});

IntStream.of(0, 2).forEach((i) -> executor.submit(() -> {
    long stamp = lock.readLock();
    try {
        System.out.println(map.get("foo"));
        sleep(1);
    } finally {
        lock.unlockRead(stamp);
    }
}));

executor.shutdown();
```

通过`readLock()`和`writeLock()`来获取读锁或写锁时会返回一个标记，
它可以在稍后用于在`finally`块中解锁。要记住`StampedLock`并没有实现重入特性。
每次调用加锁都会返回一个新的标记，并且在没有可用的锁时阻塞，即使相同线程已经拿锁了。
所以你需要额外注意不要出现死锁。

下面是一个乐观锁的例子:

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
StampedLock lock = new StampedLock();

executor.submit(() -> {
    long stamp = lock.tryOptimisticRead();
    try {
        System.out.println("是否持有乐观锁: " + lock.validate(stamp));
        sleep(1);
        System.out.println("是否持有乐观锁: " + lock.validate(stamp));
        sleep(2);
        System.out.println("是否持有乐观锁: " + lock.validate(stamp));
    } finally {
        lock.unlock(stamp);
    }
});

executor.submit(() -> {
    long stamp = lock.writeLock();
    try {
        System.out.println("获得写锁");
        sleep(2);
    } finally {
        lock.unlock(stamp);
        System.out.println("写完毕");
    }
});

executor.shutdown();
```

乐观的读锁通过调用tryOptimisticRead()获取，它总是返回一个标记而不阻塞当前线程，无论锁是否真正可用。如果已经有写锁被拿到，返回的标记等于0。你需要总是通过lock.validate(stamp)检查标记是否有效。

执行上面的代码会产生以下输出：

```bash
是否持有乐观锁: true
获得写锁
是否持有乐观锁: false
写完毕
是否持有乐观锁: false
```

乐观锁在刚刚拿到锁之后是有效的。和普通的读锁不同的是，乐观锁不阻止其他线程同时获取写锁。
在第一个线程暂停一秒之后，第二个线程拿到写锁而无需等待乐观的读锁被释放。此时，乐观的读锁就不再有效了。
甚至当写锁释放时，乐观的读锁还处于无效状态。

所以在使用乐观锁时，你需要每次在访问任何共享可变变量之后都要检查锁，来确保读锁仍然有效。

## Future

Future接口在JDK1.5种被引入，一般我们使用它以异步的方式执行一个耗时的操作。

```java
ExecutorService executor = Executors.newCachedThreadPool();

Future<Boolean> future = executor.submit(() -> {
    sleep(5);
    System.out.println("执行完毕");
    return true;
});

try {
    Boolean result = future.get(10, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    // 当前线程在等待过程中被中断
} catch (ExecutionException e) {
    // 计算抛出一个异常
} catch (TimeoutException e) {
    // 在Future对象完成之前超过已过期
}
executor.shutdown();
```

这种方式我们开启一个线程异步去执行耗时的操作，可以调用`Future`的get方法
来获取结果，如果操作已经完成，该方法会立刻返回操作的结果，否则它会阻塞你的线程，直到操作完成，返回相应的结果。

这样的场景有什么问题吗？如果长时间都没有结果返回，就会一直等待，上面的例子中
我们使用了一个带有超时参数的get方法，这样不会让你永远等待下去。

虽然`Future`提供了异步编程的能力，但是获取结果不是很方便，必须通过阻塞和轮询的方式得到任务的结果。
阻塞的方式违背了异步编程的初衷，轮询会耗费多余的CPU资源，也无法及时得到计算结果。

在Java8中, 新增 `CompletableFuture`，提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，
提供了函数式编程的能力，可以通过回调的方式处理计算结果，并且提供了转换和组合CompletableFuture的方法。

我们来修改一下上面的代码：

```java
CompletableFuture<Boolean> completableFuture = CompletableFuture.supplyAsync(() -> {
    sleep(5);
    System.out.println("执行完毕");
    return true;
});

try {
    boolean result = completableFuture.get();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```

尽管Future可以代表在另外的线程中执行的一段异步代码，但是你还是可以在本身线程中执行：


参考资料：

- [一次聊天引发的思考--java并发包](http://www.cnblogs.com/davidwang456/p/4670777.html)
