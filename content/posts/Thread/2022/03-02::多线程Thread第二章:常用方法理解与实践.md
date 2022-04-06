+++
title = "多线程Thread第二章：常用方法理解与实践"
date = "2022-03-02 23:21:52"
url = "archives/thread/2"
tags = ["Thread"]
categories = ["后端","Thread多线程学习"]
featuredImage = "https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220405/5948737d287244259ae370d2bbbfe262.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10"
+++

## 常见方法

|       方法名        | static |                           功能说明                           |                                                           注意                                                           |
|:----------------:|:------:|:--------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------:|
|     start()      |        |                  启动一个新的线程，在新的线程中运行run方法                  |     start()方法只是让当前线程进入就绪状态，也可能立即执行，但这取决于`CPU时间片`是否分给了它。start()方法在新的线程中只能调用一次，如果多次调用会报IllegalThreadStateException异常     |
|      run()       |        |                       新线程启动后会调用的方法                       |                如果在构造Thread对象时传递了Runnable接口参数，则启动线程时会执行Runnable的run方法。否则不进行其它操作。但也可以构造Thread对象子类来重写run方法                |
|      join()      |        | 等待线程运行结束。比如有一个子线程耗时长，主线程需要子线程的结果，那么主线程可以使用join来等待子线程运行结束 ||
|   join(long n)   |        |                     等待线程运行结束，最多等待n毫秒                     ||
|     getId()      |        |                        获取线程长整型的ID                        |                                                          ID唯一                                                          |
|    getName()     |        |                          获取线程名                           ||
| setName(String)  |        |                          修改线程名                           ||
|  getPriority()   |        |                         获取线程优先级                          ||
| setPriority(int) |        |                         修改线程优先级                          |                                   java中规定线程优先级是1-10的证书，较大的优先级能提高该线程被CPU调度的几率，但不是绝对的                                    |
|    getState()    |        |                          获取线程状态                          | Java中线程状态是用6个枚举表示，分别是：`NEW(创建)`、`RUNNABLE(可运行的)`、`BLOCKED(阻塞)`、`WAITING(等待)`、`TIMED_WAITING(带有时间的等待)`、`TERMINATED(结束)` |
| isInterrupted()  |        |                         判断是否被打断                          |                                                       不会清除`打断标记`                                                       |
|    isAlive()     |        |                     线程是否存活（还没有运行完毕）                      ||
|   interrupt()    |        |                           打断线程                           |    如果被打断线程正在sleep、wait、join会导致被打断的线程抛出interruptedException,并清除`打断标记`；如果打断的正在运行的线程，则会设置`打断标记`；park的线程被打断，也会设置`打断标记`     |
|  interrupted()   | static |                       判断当前线程是否被打断                        |                                                       会清除`打断标记`                                                        |
| currentThread()  | static |                       获取当前正在执行的线程                        ||
|  sleep(long n)   | static |             让当前执行的线程休眠n毫秒，休眠时让出CPU的时间片给其它线程              ||
|     yield()      | static |                   提示线程调度器让出当前线程对CPU的使用                   |                                                       主要是为了测试和调试                                                       |
  

### start 与 run

- 直接调用run是在主线程中执行了run方法，没有启动新的线程
- 使用start是启动了新的线程，通过新的额线程间接执行了run方法中的代码

### sleep 与 yield

### sleep

1、调用了sleep会让当前线程从`running`进入`timed waiting`状态（阻塞）

2、其它线程可以使用 `interrupt`方法打断正在睡眠的线程，这时sleep方法会抛出`InterruptedException`

3、睡眠结束后的线程未必会立刻得到执行

4、建议用`TimeUnit`的sleep代替Thread的sleep来获得更好的可读性

### yield

1、调用yield会让当前线程从`running`进入`runnable`就绪状态，然后调度执行其它线程

2、具体的实现依赖于操作系统的任务调度器

### 线程优先级

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果CPU比较忙，那么优先极高的线程会获得更多的时间片，但CPU闲时，优先级几乎没有作用

### interrupt方法详解

1、打断sleep、wait、join的线程。这几个方法都会让线程进入阻塞状态，打断sleep的线程，会清空打断状态，以sleep为例

```java
public static void main(String[]args){
    test1();        
}

private static void test1() throws InterruptedException {
    Thread t1 = new Thread(()->{
        sleep(1);
        }, "t1");
    t1.start();
    sleep(0.5);
    t1.interrupt();
    log.debug(" 打断状态: {}", t1.isInterrupted());
}
```
输出

```
java.lang.InterruptedException: sleep interrupted
 at java.lang.Thread.sleep(Native Method)
 at java.lang.Thread.sleep(Thread.java:340)
 at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
 at cn.itcast.n2.util.Sleeper.sleep(Sleeper.java:8)
 at cn.itcast.n4.TestInterrupt.lambda$test1$3(TestInterrupt.java:59)
 at java.lang.Thread.run(Thread.java:745)
21:18:10.374 [main] c.TestInterrupt - 打断状态: false
```

2、打断正常运行的线程

打断正常运行的线程，不会清空打断状态

```java
private static void test2() throws InterruptedException {
    Thread t2 = new Thread(()->{
        while(true) {
            Thread current = Thread.currentThread();
            boolean interrupted = current.isInterrupted();
            if(interrupted) {
                log.debug(" 打断状态: {}", interrupted);
                break;
             }
         }
     }, "t2");
    t2.start();
    sleep(0.5);
    t2.interrupt();
}
```

输出

```
20:57:37.964 [t2] c.TestInterrupt - 打断状态: true
```

### 终止模式之两阶段终止模式

在一个线程T1中如何"优雅"终止线程T2？这里的【优雅】指的是给 T2 一个料理后事的机会。

#### 1、错误的思路

- 使用线程对象的stop()方法停止线程

    - stop方法会真正的杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其它的线程将永远无法获取锁

- 使用System.exit(int)方法停止线程

    - 目的仅是停止一个线程，但这种做法会让整个程序都停止

#### 2、两阶段终止模式

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220405/f8a8e3ed31814542bb6930a0db0988ca.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

##### 2.1、利用isInterrupted

interrupt可以打断正在执行的线程，无论这个线程是在sleep、wait还是正常运行

```java
class TPTInterrupt {
    private Thread thread;
    public void start(){
      thread = new Thread(() -> {
        while(true) {
          Thread current = Thread.currentThread();
          if(current.isInterrupted()) {
            log.debug("料理后事");
            break;
          }
          try {
            Thread.sleep(1000);
            log.debug("将结果保存");
          } catch (InterruptedException e) {
            current.interrupt();
          }
          // 执行监控操作
        }},"监控线程");
      thread.start();
    }
    
    public void stop() {
        thread.interrupt();
    }
}
```

调用

```java
TPTInterrupt t = new TPTInterrupt();
t.start();
Thread.sleep(3500);
log.debug("stop");
t.stop();
```

结果

```
11:49:42.915 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:43.919 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:44.919 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:45.413 c.TestTwoPhaseTermination [main] - stop 
11:49:45.413 c.TwoPhaseTermination [监控线程] - 料理后事
```

### 打断park线程

打断park线程，不会清空打断状态

```java
private static void test3() throws InterruptedException {
  Thread t1 = new Thread(() -> {
    log.debug("park...");
    LockSupport.park();
    log.debug("unpark...");
    log.debug("打断状态：{}", Thread.currentThread().isInterrupted());
   }, "t1");
  t1.start();
  sleep(0.5);
  t1.interrupt();
}
```

输出

```
21:11:52.795 [t1] c.TestInterrupt - park... 
21:11:53.295 [t1] c.TestInterrupt - unpark... 
21:11:53.295 [t1] c.TestInterrupt - 打断状态：true
```

`如果打断标记已经是 true, 则 park 会失效。`可以使用`Thread.interrupted()`清除打断状态

### 不推荐的方法

还有一些不推荐使用的方法，这些方法已过时，容易破坏同步代码块，照成线程死锁


|    方法名    | static |    功能说明    | 
|:---------:|:------:|:----------:|
|  stop()   |        |   停止线程运行   |
| suspend() |        | 挂起（暂停）线程运行 | 
| resume()  |        |   恢复线程运行   |

### 主线程与守护线程

默认情况下，Java进程需要等待所有线程都运行结束才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，
即使守护线程的代码没有执行完，也会强制结束。

```java
log.debug("开始运行1...");
Thread t1 = new Thread(() -> {
    log.debug("开始运行2...");
    sleep(2);
    log.debug("运行结束3...");
}, "daemon");
// 设置该线程为守护线程
t1.setDaemon(true);
t1.start();
sleep(1);
log.debug("运行结束4...");
```
输出

```
08:26:38.123 [main] c.TestDaemon - 开始运行1... 
08:26:38.213 [daemon] c.TestDaemon - 开始运行2... 
08:26:39.215 [main] c.TestDaemon - 运行结束4...
```

**注意**

- 垃圾回收器线程就是一种守护线程
- Tomcat中的Acceptor和Poller线程都是守护线程，所以Tomcat接收到shutdown命令后，不会等
  待它们处理完当前请求

### 五种状态

这是从**操作系统**层面来描述的

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220405/18c0778367dd405bb2832c1c82596c67.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

- 【初始化状态】仅是在语言层面上创建了线程对象，还位于操作系统线程关联
- 【可运行状态】（就绪状态）指该线程已被创建（与操作系统线程关联），可以由CPU调度执行
- 【运行状态】指获取了 CPU 时间片运行中的状态
  - 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
  - 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入
  【阻塞状态】
  - 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑
  调度它们
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态

### 六种状态

这是从Java API层面来描述的。根据`Thread.State`枚举，分为六种状态。

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220405/5948737d287244259ae370d2bbbfe262.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

- `NEW`线程刚被创建，但是还没有调用`start()`方法
- `RUNNABLE`当调用了 `start()` 方法之后，注意，Java API 层面的 `RUNNABLE` 状态涵盖了 操作系统 层面的
  【可运行状态】、【运行状态】和【阻塞状态】（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为
  是可运行）
- `BLOCKED` ， `WAITING` ， `TIMED_WAITING` 都是 Java API 层面对【阻塞状态】的细分，后面会在状态转换一节
  详述
- `TERMINATED` 当线程代码运行结束

### 小王烧水习题

想泡壶茶喝。当时的情况是：开水没有；水壶要洗，茶壶、茶杯要洗；火已生了，茶叶也有了。

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220406/de2371f1212942d993d456522ef9fffc.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

```java
package com.zhushuyong.day01;

import lombok.extern.slf4j.Slf4j;

/**
 *
 * @author zhusy
 * @since 2022/4/6
 */
@Slf4j
public class OverallPlan {

    public static void main(String[] args) {
        OverallPlan overallPlan = new OverallPlan();
        overallPlan.test1();
    }

    private void test1() {
        Thread t1 = new Thread(()->{
            log.info("洗水壶1秒钟");
            sleep(1);
            log.info("烧开水10秒钟");
            sleep(10);
        }, "烧水的老王");
        Thread t2 = new Thread(()->{
            log.info("洗茶壶1秒钟");
            sleep(1);
            log.info("洗茶杯2秒钟");
            sleep(2);
            log.info("拿茶叶1秒钟");
            sleep(1);
            try {
                t1.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("泡茶1秒钟");
            sleep(1);
        }, "做杂事的小王");
        t1.start();
        t2.start();
    }

    private void sleep(int time) {
        try {
            Thread.sleep(time * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

输出
```
/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55995:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/zhusy/IdeaProjects/hello-java/thread/target/classes:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-databind/2.10.0/jackson-databind-2.10.0.jar:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-annotations/2.10.0/jackson-annotations-2.10.0.jar:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-core/2.10.0/jackson-core-2.10.0.jar:/usr/local/maven-repository/org/springframework/spring-context/5.2.0.RELEASE/spring-context-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-aop/5.2.0.RELEASE/spring-aop-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-beans/5.2.0.RELEASE/spring-beans-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-core/5.2.0.RELEASE/spring-core-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-jcl/5.2.0.RELEASE/spring-jcl-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-expression/5.2.0.RELEASE/spring-expression-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-webmvc/5.2.0.RELEASE/spring-webmvc-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-web/5.2.0.RELEASE/spring-web-5.2.0.RELEASE.jar:/usr/local/maven-repository/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar:/usr/local/maven-repository/ch/qos/logback/logback-core/1.2.3/logback-core-1.2.3.jar:/usr/local/maven-repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/usr/local/maven-repository/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar:/usr/local/maven-repository/org/projectlombok/lombok/1.18.22/lombok-1.18.22.jar com.zhushuyong.day01.OverallPlan
22:00:03.123 [做杂事的小王] INFO com.zhushuyong.day01.OverallPlan - 洗茶壶1秒钟
22:00:03.124 [烧水的老王] INFO com.zhushuyong.day01.OverallPlan - 洗水壶1秒钟
22:00:04.130 [烧水的老王] INFO com.zhushuyong.day01.OverallPlan - 烧开水10秒钟
22:00:04.130 [做杂事的小王] INFO com.zhushuyong.day01.OverallPlan - 洗茶杯2秒钟
22:00:06.135 [做杂事的小王] INFO com.zhushuyong.day01.OverallPlan - 拿茶叶1秒钟
22:00:14.135 [做杂事的小王] INFO com.zhushuyong.day01.OverallPlan - 泡茶1秒钟

Process finished with exit code 0
```

**这里重点其实就是用到了`join`的特性，只有当`t1.join();`运行完成后，泡茶才会往下继续走**