+++
title = "多线程Thread第一章：入门理念学习"
date = "2022-02-08 14:21:52"
url = "archives/thread/1"
tags = ["Thread"]
categories = ["后端","Thread多线程学习"]
featuredImage = "https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220217/158f99d96eff4ff09f6aca1ceeb3df6e.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10"
+++

## 进程与线程 ##

### 进程 ###

- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程
就是用来加载指令、管理内存、管理IO的
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
- 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器等），也有的程序只能启动一个实例进程（例如网易云音
乐、360安全卫士等）

### 线程 ###
- 一个进程之内可以分为一到多个线程。
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给CPU执行
- Java中，线程作为最小的调度单位，进程作为资源分配的最小单位。在window中进程是不活动的，只是作为线程的容器

### 二者对比 ###

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集
- 进程拥有共享的资源，比如内存空间等，供其内部的线程共享
- 进程间通信较为复杂
  - 同一台计算机的进程通信称为IPC（inter-process communication）
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如http
- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低

## 并行与并发 ##

单核CPU下，线程实际还是`串行执行`的。操作系统中有一个组件叫做任务调度器，将CPU的时间片（windows下的时间片最小约为15毫秒）分给不同的程序使用，
只是由于CPU在线程间的切换非常快，感觉是`同时运行的`。总结为一句话就是：`微观串行，宏观并行`。

一般会将这种`线程轮流使用CPU`的做法称之为`并发 concurrent`

| CPU | 时间片 1 | 时间片 2 | 时间片 3 | 时间片 4 |
|:----:|:----:|:----:|:----:|:----:|
| core | 线程 1 | 线程 2  | 线程 3 | 线程 4 |

多核CPU下，每个`核（core）`都可以调度运行线程，这时候线程可以是并行的。

|  CPU  | 时间片 1 | 时间片 2 | 时间片 3 | 时间片 4 |
|:-----:|:----:|:----:|:----:|:----:|
| core1 | 线程 1 | 线程 2  | 线程 3 | 线程 4 |
| core2 | 线程 1 | 线程 2  | 线程 3 | 线程 4 |

### 异步调用 ###

以调用方角度来讲，如果
* 需要等待结果返回，才能继续运行就是同步
* 不需要等待结果返回，就能继续运行就是异步

```java
package com.zhushuyong.day01;

import java.util.Arrays;
import java.util.concurrent.FutureTask;

/**
 * 线程的串行与并行的区别
 * @author zhusy
 * @since 2022/2/18
 */
public class SerialAndParallel {

  public static void main(String[] args) throws Exception {
    long statTime = System.currentTimeMillis();
    System.out.println("开始执行时间：" + statTime);
    //System.out.println(serial());
    System.out.println(parallel());
    long endTime = System.currentTimeMillis();
    System.out.println("结束时间：" + endTime);
    System.out.println("相差毫秒数：" + (endTime - statTime));
  }

  /**
   * 先定义一个int类型数组，空间是一亿的长度
   */
  static int[] ARRAY = new int[100000000];

  /**
   * 静态代码块，在JVM虚拟机加载类的时候就会执行，且只会执行一次
   */
  static {
    //赋值一亿个等于1的值到数组中
    Arrays.fill(ARRAY, 1);
  }

  /**
   * 并行执行一亿个数相加。开启4个线程计算，每个线程计算25000000个数
   * @return
   * @throws Exception
   */
  public static int parallel() throws Exception {
    int[] array = ARRAY;
    FutureTask<Integer> task = new FutureTask<>(()->{
      int sum = 0;
      for (int i =0; i< 25000000; i++) {
        sum += array[i];
      }
      return sum;
    });
    FutureTask<Integer> task1 = new FutureTask<>(()->{
      int sum = 0;
      for (int i=0; i< 25000000; i++) {
        sum += array[25000000 + i];
      }
      return sum;
    });
    FutureTask<Integer> task2 = new FutureTask<>(()->{
      int sum = 0;
      for (int i=0; i< 25000000; i++) {
        sum += array[50000000 + i];
      }
      return sum;
    });
    FutureTask<Integer> task3 = new FutureTask<>(()->{
      int sum = 0;
      for (int i=0; i<25000000; i++) {
        sum += array[75000000 + i];
      }
      return sum;
    });
    new Thread(task).start();
    new Thread(task1).start();
    new Thread(task2).start();
    new Thread(task3).start();
    return task.get() + task1.get() + task2.get() + task3.get();
  }
  /**
   * 串行执行一亿个数相加，查看需要花费的时间
   * @return
   * @throws Exception
   */
  public static int serial() throws Exception {
    int[] array = ARRAY;
    FutureTask<Integer> task = new FutureTask<>(()->{
      int sum = 0;
      for (int i = 0; i< array.length; i++) {
        sum += array[0 + i];
      }
      return sum;
    });
    new Thread(task).start();
    return task.get();
  }

}
```
其中 `serial` 方法表示串行执行，开启一个线程执行，执行三次后平均执行时间是55毫秒左右。
`parallel`方法并行执行， 开启4个线程运行一亿个数相加，执行三次后平均执行时间是46毫秒左右。

> 注意：需要在多核CPU才能提高效率，单核仍然是轮流执行

1、单核CPU下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同的线程轮流使用CPU，不至于一个线程总占用CPU，别的线程没法干活
2、多核CPU可以并行跑多个线程，但能否提高程序运行效率还是要分情况的
 - 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有额计算任务都能拆分（参考后文的【阿姆达尔定律】）
 - 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
3、io操作不占用CPU，只是我们一般拷贝文件使用的是【阻塞io】，这时相当于线程虽然不用CPU，但需要一直等待io结束，没能充分利用线程。所以才有后面的
【非阻塞io】和【异步io】优化

## 创建和运行线程

### 使用thread

直接定义一个Thread对象参数，使用匿名内部类重写run方法。
也可以继承thread对象，重写run方法

```java
package com.zhushuyong.day01;

import lombok.extern.slf4j.Slf4j;

/**
 * @author zhusy
 * @since 2022/2/20
 */
@Slf4j(topic = "com.zhushuyong")
public class test1 {

    public static void main(String[] args) {

        Thread thread = new Thread("子线程1"){
            @Override
            public void run() {
                log.info("子线程打印数据");
            }
        };
        thread.start();
        log.info("主线程打印数据");

    }

}
```
main方法本身就是一个线程，是启动项目的主线程，同时也被称为`守护线程`

### 使用Runnable配合Thread

把【线程】和【任务】分开
* thread代表线程
* runnable可运行的任务

```java
package com.zhushuyong.day01;

import lombok.extern.slf4j.Slf4j;

/**
 * @author zhusy
 * @since 2022/2/20
 */
@Slf4j(topic = "com.zhushuyong")
public class test2 {

    public static void main(String[] args) {

        //使用匿名内部类的方式重写run方法
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log.info("这里是可执行的任务代码");
            }
        };
        //传入thread对象的构造方法中
        Thread thread = new Thread(runnable, "子线程1");
        thread.start();

        log.info("主线程执行任务代码");
    }

}
```

### Thread类与Runnable接口的关系

```java
package com.zhushuyong.day01;

import lombok.extern.slf4j.Slf4j;

/**
 * @author zhusy
 * @since 2022/2/20
 */
@Slf4j(topic = "com.zhushuyong")
public class test2 {

    public static void main(String[] args) {
        //a1方法是直接用thread的匿名内部类重写run方法
        a1();
        //a2方法使用runnable的匿名内部类重写run方法
        a2();
    }
    
    public static void a1() {
        Thread thread = new Thread(){
            @Override
            public void run() {
                log.info("可执行的任务代码");
            }
        };
        thread.start();
    }

    public static void a2() {

        //使用匿名内部类的方式重写run方法
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log.info("这里是可执行的任务代码");
            }
        };
        //传入thread对象的构造方法中
        Thread thread = new Thread(runnable, "子线程1");
        thread.start();

        log.info("主线程执行任务代码");
    }

}
```
#### 方法a2的原理
1、我们查看a2方法中的`Thread thread = new Thread(runnable, "子线程1");`的构造方法源码。
跟踪第一个Runnable target的`target`参数。进入this方法，再跟进下一个this方法引用中。

![跟踪第一步构造方法](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220220/7d0dee9f4e814f55b804cf7561b81188.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

![跟踪第二步构造方法](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220220/074e14a24ed54613bb7bc67df3568ffa.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

一直找到如下方法：
```java
 private Thread(ThreadGroup g, Runnable target, String name,
                   long stackSize, AccessControlContext acc,
                   boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security manager doesn't have a strong opinion
               on the matter, use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(
                        SecurityConstants.SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        this.tid = nextThreadID();
    }
```
2、其中`this.target = target;`这一步表示传入的target参数赋值给了thread中的一个`成员变量`。那么这个成员变量在哪用到了？
依然阅读Thread类源码找到 run 方法。
```java
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
3、发现Runnable对象不为空的时候，调用Runnable接口中的run抽象方法。只不过是在Thread中进行调用。

#### 方法a1的原理

创建了一个Thread的子类对象，并重写了父类的run方法。最终以子类的 run方法的代码为主。

> 总结：不管方式1还是方式2本质上走的都是Thread的run方法

### FutureTask配合Thread

```java
package com.zhushuyong.day01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @author zhusy
 * @since 2022/2/20
 */
@Slf4j(topic = "com.zhushuyong")
public class Test3 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
//            @Override
//            public Integer call() throws Exception {
//                log.info("执行了线程回调中的call方法");
//                //休眠（阻塞2秒）
//                Thread.sleep(2*1000);
//                return 100;
//            }
//        });
        //Java8的lambda的写法
        FutureTask<Integer> futureTask = new FutureTask<>(()->{
                log.info("执行了线程回调中的call方法");
                //休眠（阻塞2秒）
                Thread.sleep(2*1000);
                return 100;
        });

        //因为FutureTask实现了RunnableFuture接口，然后RunnableFuture接口又继承了Runnable接口，所以这里可以直接传入FutureTask参数
        Thread thread = new Thread(futureTask, "t1");
        thread.start();
        log.info("线程执行的结果:{}", futureTask.get());
    }

}
```
- 查看`FutureTask`源码得知FutureTask实现了`RunnableFuture`接口，然后RunnableFuture接口又继承了`Runnable`接口。
- `Callable`接口跟`Runnable`接口很类似，但是`Callable`接口的`call`方法是有返回值以及能抛出的异常的。
`Runnable`接口的`run`方法是没有返回值的抽象方法

## 查看进程线程的方法 ##

### window ###

任务管理器可以查看进程和线程数，也可以用来杀死进程
- `tasklist`查看进程
- `taskkill`杀死进程

### linux ###

- `ps -fe`查看所有进程
- `kill`杀死进程

### JAVA
- `jps`命令查看所有Java进程

```Bash
zhusy@zhusydeMacBook-Pro day01 % jps
658 
2437 Launcher
2438 test4
2539 Jps
zhusy@zhusydeMacBook-Pro day01 % kill 2438
zhusy@zhusydeMacBook-Pro day01 % 
```
![终止运行](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220220/f9efceba82274864b3deda08f90ab021.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

## 原理之线程运行

### 栈与栈帧

Java Virtual Machine Stacks(java虚拟机栈)

JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？答案就是线程，每个线程启动后，虚拟机就会为其分配一块栈内存。
- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

![栈帧结构代码演示](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220220/822f00bab5b14d30a85d09fa0a9013af.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

- 上图是一个简单的main方法调用演示栈帧过程，其实main启动时就是一个主线程的启动。
- 随后会有`method1`和`method2`栈帧。一次方法的调用就会 在栈帧组中多出来一次显示。遵循 `先进后出`的原则。`method2`方法栈帧最后调用，调用结束后从栈帧组中移除，随后是`method1`方法栈帧。

![栈帧调用图解.png](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220221/4af70c95b15341da9054b46d1d2a0e1e.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

1、进入main方法调试后，main栈帧首先进入`栈内存`，args是一个String类型入参数组，在堆内存中会 new 一个String类型数组。对象是在堆内存中

2、随后debug单步调试进入method1，执行到method1方法时。栈内存加入`method1栈帧`。局部变量表中加入x、y参数。紧接着是method2方法进入。
栈内存加入`method2栈帧`。

3、栈内存中还有个`程序计数器`，用来记录执行到代码的位置。

4、当`method2栈帧`执行完成之后，从栈内存移除（这里用了虚线表示，表示执行完就会移除掉）。演示`先进后出`的原则。

### 线程上下文切换

因为以下一些原因导致CPU不再执行当前的线程，转而执行另一个线程的代码

- 线程的CPU时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行
- 线程自己调用了
  - `sleep（睡眠、阻塞）`：睡眠、阻塞
  - `yield`：yield 即 "谦让"，也是 Thread 类的方法。它让掉当前线程 CPU 的时间片，使正在运行中的线程重新变成就绪状态，并重新竞争 CPU 的调度权。它可能会获取到，也有可能被其他线程获取到
  - `wait`：等待
  - `join`：主线程等待子线程的终止。也就是说主线程的代码块中，如果碰到了t.join()方法，此时主线程需要等待（阻塞），等待子线程结束了(Waits for this thread to die.),才能继续执行t.join()之后的代码块
  - `park`
  - `synchronized`
  - `lock`
  
  等方法。

当Context Switch发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java中对应的概念就是程序计数器（Program Counter Register）,
他的作用是记住下一条JVM指令的执行地址，是线程私有的
- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，入局部变量、操作数栈、返回地址等
- Context Switch 频繁发生会影响性能






