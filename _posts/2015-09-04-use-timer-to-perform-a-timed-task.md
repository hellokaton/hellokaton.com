---
layout: post
title: 使用Timer执行定时任务
categories: java
tags: java timer task
---

### 一、Timer概述

在Java开发中，会碰到一些需要定时或者延时执行某些任务的需求，这时，我们可以使用Java中的Timer类实现。

<!-- more -->

### 二、Timer介绍

```bash
Timer是一个定时器类，通过该类可以为指定的定时任务进行配置，所在jar包路径：java.util.Timer
Timer定时器实例有多种构造方法：
Timer() // 创建一个新计时器
Timer(boolean isDaemon) //创建一个新计时器，可以指定其相关的线程作为守护程序运行
Timer(String name) //创建一个新计时器，其相关的线程具有指定的名称
Timer(String name, boolean isDaemon) //创建一个新计时器，其相关的线程具有指定的名称，并且可以指定作为守护程序运行
TimerTask类是一个定时任务类，该类实现了Runnable接口，而且是一个抽象类，所在jar包路径：java.util.TimerTask
// 可以通过继承该类，来实现自己的定时任务。
public abstract class TimerTask implements Runnable
```

### 三、Timer常用方法

在特定时间执行任务，只执行一次

```java
public void schedule(TimerTask task,Date time)
```

在特定时间之后执行任务，只执行一次

```java
public void schedule(TimerTask task, long delay)
```

指定第一次执行的时间，然后按照间隔时间，重复执行

```java
public void schedule(TimerTask task, Date firstTime, long period)
```

在特定延迟之后第一次执行，然后按照间隔时间，重复执行

```java
public void schedule(TimerTask task, long delay, long period)
/*参数说明
 delay： 延迟执行的毫秒数，即在delay毫秒之后第一次执行
 period：重复执行的时间间隔
*/
```

第一次执行之后，特定频率执行，与3效果相同

```java
public void scheduleAtFixedRate(TimerTask task, Date firstTime, long period)
```

在delay毫秒之后第一次执行，后按照特定频率执行

```java
public void scheduleAtFixedRate(TimerTask task, long delay, long period)
```

Timer注销

```java
timer.cancel();
```

### schedule()和scheduleAtFixedRate()的区别

`schedule()` 方法更注重保持间隔时间的稳定：保障每隔period时间可调用一次
`scheduleAtFixedRate()` 方法更注重保持执行频率的稳定：保障多次调用的频率趋近于period时间，如果任务执行时间大于period，会在任务执行之后马上执行下一次任务

### 四、Timer使用示例

使用Timer每隔2秒打印一次数据，并且任务在Timer启动1秒之后开始

```java
import java.util.Timer;
import java.util.TimerTask;

public class TestTimer {

    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new MyTask(), 1000, 2000);
    }

}

class MyTask extends TimerTask {

    @Override
    public void run() {
        System.out.println("每隔2秒我就出现一次……");
    }

}
```

使用Timer每隔一段时间随机生成数字

```java
import java.util.Timer;
import java.util.Random;
import java.util.TimerTask;

public class TimerTest {

    public static void main(String[] args) {
        Timer timer = new Timer();
        NewTimerTask timerTask = new NewTimerTask();
        //程序运行后立刻执行任务，每隔100ms执行一次
        timer.schedule(timerTask, 0, 100);
    }

}

class NewTimerTask extends TimerTask {

    @Override
    public void run() {
        createRandomNumber();
        createRandomNumberFromMathRandom();
    }

    //用纯Math中的方法来随机生成1-10之间的随机数
    private void createRandomNumberFromMathRandom() {
        int j = (int)(Math.round(Math.random()*10 + 1));
        System.out.println("随机生成的数字为:"+j);
    }

    //用Random类的方式来随机生成1-10之间的随机数
    private void createRandomNumber() {
         Random random = new Random(System.currentTimeMillis());
         int value = random.nextInt();
         value = Math.abs(value);
         value = value%10 + 1;
         System.out.println("新生成的数字为:" + value);
    }

}
```

### 五、小结

通过上面的两个简单示例，我们可以很清楚的知道Timer的用法：

实现TimerTask接口，并即为单元任务，我们的单次运行业务逻辑写在这里面
实例化一个Timer对象，用于启动TimerTask任务，并通过调用不同的方法设置任务的执行时间、频率
在实际的应用中，Timer多用于在夜间处理比较耗时并且数据状态稳定时候的一些后台操作，例如数据统计、数据备份等。
