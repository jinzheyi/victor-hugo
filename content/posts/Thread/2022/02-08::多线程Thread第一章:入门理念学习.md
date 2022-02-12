+++
title = "多线程Thread第一章：入门理念学习"
date = "2022-02-08 14:21:52"
url = "archives/1"
tags = ["Thread"]
categories = ["后端","Thread多线程学习"]
featuredImage = "http://121.43.32.181:9000/blog/images/20220212/563aeba635eb4d7eba4b3226421adb14.png"
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
package com.zhushuyong;

import org.openjdk.jmh.annotations.*;

import java.util.Arrays;
import java.util.concurrent.FutureTask;

@Fork(1)
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations=3)
@Measurement(iterations=5)
public class MyBenchmark {
    static int[] ARRAY = new int[1000_000_00];
    static {
        Arrays.fill(ARRAY, 1);
    }
    @Benchmark
    public int c() throws Exception {
        int[] array = ARRAY;
        FutureTask<Integer> t1 = new FutureTask<>(()->{
            int sum = 0;
            for(int i = 0; i < 250_000_00;i++) {
                sum += array[0+i];
            }
            return sum;
        });
        FutureTask<Integer> t2 = new FutureTask<>(()->{
            int sum = 0;
            for(int i = 0; i < 250_000_00;i++) {
                sum += array[250_000_00+i];
            }
            return sum;
        });
        FutureTask<Integer> t3 = new FutureTask<>(()->{
            int sum = 0;
            for(int i = 0; i < 250_000_00;i++) {
                sum += array[500_000_00+i];
            }
            return sum;
        });
        FutureTask<Integer> t4 = new FutureTask<>(()->{
            int sum = 0;
            for(int i = 0; i < 250_000_00;i++) {
                sum += array[750_000_00+i];
            }
            return sum;
        });
        new Thread(t1).start();
        new Thread(t2).start();
        new Thread(t3).start();
        new Thread(t4).start();
        return t1.get() + t2.get() + t3.get()+ t4.get();
    }
    @Benchmark
    public int d() throws Exception {
        int[] array = ARRAY;
        FutureTask<Integer> t1 = new FutureTask<>(()->{
            int sum = 0;
            for(int i = 0; i < 1000_000_00;i++) {
                sum += array[0+i];
            }
            return sum;
        });
        new Thread(t1).start();
        return t1.get();
    }
}
```
其中 `@BenchmarkMode(Mode.AverageTime)` 表示测试模式（统计测试的平均时间）。
`@Warmup(iterations=3)`表示热身3次，程序在JVM虚拟机中头一次执行时间会偏长，多次运行后，JVM才能比较准确的运行程序真正需要的时间。
`@Measurement(iterations=5)`表示进行5轮测试





