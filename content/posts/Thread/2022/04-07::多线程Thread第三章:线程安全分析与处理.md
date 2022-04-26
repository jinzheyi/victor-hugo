+++
title = "多线程Thread第三章：线程安全分析与处理"
date = "2022-04-07 23:28:52"
url = "archives/thread/3"
tags = ["Thread"]
categories = ["后端","Thread多线程学习"]
featuredImage = "https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/7d34070eaf7a45b1a4dea0f463c3b607.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10"
+++

## 引用

**线程共享内存模型** 顾名思义就是通过共享内存来实现并发的模型，当多个线程在并发执行中使用共享资源时不对所共享的资源进行约定或特殊处理时就会出现读到脏数据、
无效的数据等问题；而为了解决共享资源所引起的这些问题，Java中引入了同步(synchronized)、锁(lock)、原子类型(atomic)等这些用于处理共享资源的操作；

## 内存共享模型问题

现在我们看下面的一个例子：

```java
package com.zhushuyong.day01;

/**
 * @author zhusy
 * @since 2022/4/8
 */
public class ShareModel {

    //定义一个共享资源
    static int count = 0;
    
    public static void main(String[] args) throws InterruptedException{
        Thread t1 = new Thread(()-> {
            for (int i=0; i<=5000; i++) {
                count++;
            }
        }, "小明做加法");

        Thread t2 = new Thread(()-> {
            for (int i=0; i<=5000; i++) {
                count--;
            }
        }, "小红做减法");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("count==" + count);
    }

}
```

这里我们定义了两个线程来操作同一个共享资源数据，程序的本意是让它最终的结果运行是等于0；但是多运行几次，很少的情况下会等于0，大部分时候结果是随机的，有正数结果也会有负数结果。
那么照成这种问题的原因是什么了？因为 Java 中对静态变量的自增，自减并不是原子操作，要彻底理解，必须从字节码来进行分析。 例如对于 i++ 而言（i 为静态变量），实际会产生如下的 JVM 字节码指令：

- getstatic i // 获取静态变量i的值
- iconst_1 // 准备常量1
- iadd // 自增
- putstatic i // 将修改后的值存入静态变量i

而对应 i-- 也是类似：

- getstatic i // 获取静态变量i的值
- iconst_1 // 准备常量1
- isub // 自减
- putstatic i // 将修改后的值存入静态变量i

而Java的内存模型如下，完成静态变量的自增、自减需要在主存和工作内存中进行数据交换：

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220408/8b0ccf6207d64e5286f0d04f4be2d4ff.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

如果是单线程的话，上面的代码是按顺序执行的，不会有问题。我们用交互图罗列下代码的过程

出现负数：

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220408/1ba50a9a8b514eb68cf7123fca8c7b8e.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

出现正数：

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220408/5a24aa535c5f406da7dd749d7d9f0324.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

`其实本质上还是线程CPU时间片用完，发生了上下文切换。当其中一个线程计算到一半的时候时间片恰好用完，这时候上下文切换。另一个
线程介入，这时候结果就容易出现与预期的不一致偏差`

### 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的，出现问题的情况是在多个线程访问**`共享资源`**

  - 多个线程读`共享资源`其实也没问题
  - 但在多个线程对`共享资源`读写操作时就会发生指令交错，就会出现问题
  
- 一段代码块内如果存在对`共享资源`的多线程读写操作，称这段代码块为`临界区`

例如上述的代码中

```java
count++;  //临界区
```
以及
```java
count--;  //临界区
```

### 竞态条件 Race Condition

多个线程在临界区内执行，由于代码的`执行序列不同`而导致结果无法预测，称之为发生了`竞态条件`

## Synchronized解决方案

为了避免临界区的竞态条件发生，有多种手段可以达到目的。

- 阻塞式的解决方案：synchronized, Lock
- 非阻塞式的解决方案：原子变量

`synchronized`俗称【对象锁】，它采用`互斥`的方式让同一时刻至多只有一个线程能持有【对象锁】，其它
线程再想拿到这个【对象锁】时就会发生阻塞。这样就能保证拥有锁的线程可以安全的执行`临界区`内的代码，不用担心线程发生了
上下文切换却没有完成线程当前的动作的情况发生

**注意**

虽然Java中互斥和同步都可以采用`synchronized`关键字来完成，但区别还是有的：
- 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区的代码
- 同步是由于线程执行的先后，顺序不同，需要一个线程等待其它线程运行到某个点

### synchronized的语法

**语法**

```java
synchronized(对象) //线程1， 线程2（blocked）
{
    临界区    
}
```
synchronized解决代码如下：
```java
package com.zhushuyong.day01;

/**
 * @author zhusy
 * @since 2022/4/8
 */
public class ShareModel {

    //定义一个共享资源
    static int count = 0;

    static final Object obj = new Object();

    public static void main(String[] args) throws InterruptedException{
        Thread t1 = new Thread(()-> {
            for (int i=0; i<=5000; i++) {
                synchronized (obj) {
                    count++;
                }
            }
        }, "小明做加法");

        Thread t2 = new Thread(()-> {
            for (int i=0; i<=5000; i++) {
                synchronized (obj) {
                    count--;
                }
            }
        }, "小红做减法");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("count==" + count);
    }

}
```

用图来表示

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220408/232b98bc80964326834ed0559f848290.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)


synchronized 实际是用对象锁保证了`临界区`内代码的原子性，临界区内的代码对外是不可分割的，不会被线程切
换所打断。

问题

- 如果把 synchronized(obj) 放在 for 循环的外面，如何理解？-- 能达到预期的结果。原子性
- 如果 t1 synchronized(obj1) 而 t2 synchronized(obj2) 会怎样运作？-- 不能达到预期的结果。锁对象
- 如果 t1 synchronized(obj) 而 t2 没有加会怎么样？如何理解？-- 不能达到预期的结果，只对一个线程操作加了锁。锁对象

`类只有一个，new出来的实例可以有多个。类对象在内存中只有一份，是单例。类对象!=new出来的实例对象`

### 所谓的“线程八锁”

其实就是考察 synchronized 锁住的是哪个对象

情况1：12 或 21

```java
class Number{
  public synchronized void a() {
      log.debug("1");
  }
  public synchronized void b() {
      log.debug("2");
  }
}
public static void main(String[] args) {
    Number n1 = new Number();
    new Thread(()->{ n1.a(); }).start();
    new Thread(()->{ n1.b(); }).start();
}
```

情况2：1s后12，或 2 1s后 1

```java
class Number{
  public synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public synchronized void b() {
    log.debug("2");
  }
}
public static void main(String[] args) {
    Number n1 = new Number();
    new Thread(()->{ n1.a(); }).start();
    new Thread(()->{ n1.b(); }).start();
}
```

情况3：3 1s 12 或 23 1s 1 或 32 1s 1

```java
class Number{
  public synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public synchronized void b() {
    log.debug("2");
  }
  public void c() {
    log.debug("3");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n1.b(); }).start();
  new Thread(()->{ n1.c(); }).start();
}
```

情况4：2 1s 后 1

```java
class Number{
  public synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public synchronized void b() {
    log.debug("2");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  Number n2 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n2.b(); }).start();
}
```

情况5：2 1s 后 1

```java
class Number{
  public static synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public synchronized void b() {
    log.debug("2");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n1.b(); }).start();
}
```

情况6：1s 后12， 或 2 1s后 1

```java
class Number{
  public static synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public static synchronized void b() {
    log.debug("2");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n1.b(); }).start();
}
```

情况7：2 1s 后 1

```java
class Number{
  public static synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public synchronized void b() {
      log.debug("2");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  Number n2 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n2.b(); }).start();
}
```

情况8：1s 后12， 或 2 1s后 1

```java
class Number{
  public static synchronized void a() {
    sleep(1);
    log.debug("1");
  }
  public static synchronized void b() {
    log.debug("2");
  }
}
public static void main(String[] args) {
  Number n1 = new Number();
  Number n2 = new Number();
  new Thread(()->{ n1.a(); }).start();
  new Thread(()->{ n2.b(); }).start();
}
```

## 变量的线程安全分析

- **成员变量和静态变量是否线程安全**

  - 如果它们没有共享，则线程安全
  - 如果它们被共享了，根据它们的状态是否能够改变，又分为两种情况
    
    - 如果只有读操作，则线程安全
    - 如果有读写操作、这段代码是临界区，需要考虑线程安全
    
- **局部变量是否线程安全**

  - 局部变量是线程安全的
  - 但局部变量引用的对象则未必
    
    - 如果该对象没有逃离方法的作用访问，它是线程安全的
    - 如果该对象逃离了方法的作用范围，需要考虑线程安全

### 局部变量线程安全分析

线程调用方法时，局部变量会在每个线程的栈帧内存中被创建多份，因此不存在共享

局部变量的引用稍有不同

```java
class ThreadUnsafe {
    ArrayList<String> list = new ArrayList<>();
    public void method1(int loopNumber) {
        for (int i = 0; i < loopNumber; i++) {
            // { 临界区, 会产生竞态条件
            method2();
            method3();
            // } 临界区
         }
     }
    private void method2() {
        list.add("1");
     }
    private void method3() {
        list.remove(0);
     }
}


static final int THREAD_NUMBER = 2;
static final int LOOP_NUMBER = 200;
public static void main(String[] args) {
    ThreadUnsafe test = new ThreadUnsafe();
    for (int i = 0; i < THREAD_NUMBER; i++) {
        new Thread(() -> {
            test.method1(LOOP_NUMBER);
         }, "Thread" + i).start();
     }
}
```

其中一种情况是，如果线程2还未add，线程1 remove就会报错：
```
Exception in thread "Thread1" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0 
 at java.util.ArrayList.rangeCheck(ArrayList.java:657) 
 at java.util.ArrayList.remove(ArrayList.java:496) 
 at cn.itcast.n6.ThreadUnsafe.method3(TestThreadSafe.java:35) 
 at cn.itcast.n6.ThreadUnsafe.method1(TestThreadSafe.java:26) 
 at cn.itcast.n6.TestThreadSafe.lambda$main$0(TestThreadSafe.java:14) 
 at java.lang.Thread.run(Thread.java:748)
```

分析：

- 无论哪个线程中的 method2 引用的都是同一个对象中的 list 成员变量

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/560aef4f3e2c44bab64e48e99f551757.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)


解决方法：将list对象修改为局部变量

- list是局部变量，每个线程调用时会创建不同的实例，没有共享
- 而method2 的参数是从 method1 中传递过来的，与 method1 中引用同一个对象
- method3 的参数分析与 method2 相同

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/afca2d41c61241b99c2ddf3822625785.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

方法访问修饰符带来的思考，如果吧 method2 和 method3 的方法修改为 public 会不会代理线程安全问题？

- 情况1：有其它线程调用 method2 和 method3
- 情况2：在 情况1 的基础上，为 ThreadSafe 类添加子类，子类覆盖 method2 或 method3 方法，即

```java
package com.zhushuyong.day01;

import java.util.ArrayList;

/**
 * @author zhusy
 * @since 2022/4/17
 */
public abstract class ThreadSafe {

  static final int THREAD_NUMBER = 2;
  static final int LOOP_NUMBER = 300;
  public static void main(String[] args) {
    ThreadSafeSubClass test = new ThreadSafeSubClass();
    for (int i = 0; i < THREAD_NUMBER; i++) {
      new Thread(() -> {
        test.method1(LOOP_NUMBER);
      }, "Thread" + i).start();
    }
  }

  public final void method1(int loopNumber) {
    ArrayList<String> list = new ArrayList<>();
    for (int i = 0; i < loopNumber; i++) {
      method2(list);
      method3(list);
    }
  }

  public void method2(ArrayList<String> list){
    list.add("1");
  }

  public void method3(ArrayList<String> list) {
    list.remove(0);
  }

}

class ThreadSafeSubClass extends ThreadSafe {

  @Override
  public void method3(ArrayList<String> list) {
    new Thread(()->{
      list.remove(0);
    }).start();
  }

}
```

```
/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=58084:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/zhusy/IdeaProjects/hello-java/thread/target/classes:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-databind/2.10.0/jackson-databind-2.10.0.jar:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-annotations/2.10.0/jackson-annotations-2.10.0.jar:/usr/local/maven-repository/com/fasterxml/jackson/core/jackson-core/2.10.0/jackson-core-2.10.0.jar:/usr/local/maven-repository/org/springframework/spring-context/5.2.0.RELEASE/spring-context-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-aop/5.2.0.RELEASE/spring-aop-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-beans/5.2.0.RELEASE/spring-beans-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-core/5.2.0.RELEASE/spring-core-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-jcl/5.2.0.RELEASE/spring-jcl-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-expression/5.2.0.RELEASE/spring-expression-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-webmvc/5.2.0.RELEASE/spring-webmvc-5.2.0.RELEASE.jar:/usr/local/maven-repository/org/springframework/spring-web/5.2.0.RELEASE/spring-web-5.2.0.RELEASE.jar:/usr/local/maven-repository/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar:/usr/local/maven-repository/ch/qos/logback/logback-core/1.2.3/logback-core-1.2.3.jar:/usr/local/maven-repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/usr/local/maven-repository/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar:/usr/local/maven-repository/org/projectlombok/lombok/1.18.22/lombok-1.18.22.jar com.zhushuyong.day01.ThreadSafe
Exception in thread "Thread-599" Exception in thread "Thread-589" java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
	at java.base/java.util.Objects.checkIndex(Objects.java:359)
	at java.base/java.util.ArrayList.remove(ArrayList.java:504)
	at com.zhushuyong.day01.ThreadSafeSubClass.lambda$method3$0(ThreadSafe.java:45)
	at java.base/java.lang.Thread.run(Thread.java:833)
java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
	at java.base/java.util.Objects.checkIndex(Objects.java:359)
	at java.base/java.util.ArrayList.remove(ArrayList.java:504)
	at com.zhushuyong.day01.ThreadSafeSubClass.lambda$method3$0(ThreadSafe.java:45)
	at java.base/java.lang.Thread.run(Thread.java:833)

Process finished with exit code 0
```

> 从这个例子可以看出 private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】


### 常见线程安全类

- String
- Integer
- StringBuffer(操作字符串类)
- Random(随机数)
- Vector(集合)
- Hashtable(操作map)
- java.util.concurrent包下的类

`**这里说他们是线程安全的是指，多个线程调用他们同一个实例的某个方法时，是线程安全的，也可以理解为**`

```java
Hashtable hashtable = new Hashtable();
Thread t1 = new Thread(()->{
    hashtable.put("key1", "value1");
});

Thread t2 = new Thread(()->{
    hashtable.put("key2", "value2");
});
```

- 通过查看源码知道他们的每个方法时原子的
- 
```java
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```

> 但**注意**他们多个方法的组合不是原子的[非线程安全]

```java
Hashtable table = new Hashtable();
// 线程1，线程2
if( table.get("key") == null) {
table.put("key", value);
}
```

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/17146cdd47504ce9afb5a09e9ed6f5d6.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)


### 不可变类线程安全性

String、 Integer等都是不可变类，因为其内部的状态不可以变化，因此他们的方法都是线程安全的。

这里来分析 String 的substring的执行过程：

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/1e1b24ba36aa4fc0b0bdfedb03595a15.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/f12beeb620ee4a43ae37f7d5ff5f8ead.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220417/9c11255f1b8b47f39f3782a5afd9ac2c.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)


## Monitor原理

Monitor 被翻译为 **监视器** 或 **管程**

每个Java对象都可以关联一个Monitor对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象
的指针

Monitor 结构如下

![](https://zhushuyong.oss-cn-hangzhou.aliyuncs.com/images/20220426/1bceda941f01470f911519b8f74a0082.png?x-oss-process=image/auto-orient,1/interlace,1/quality,q_50/format,jpg/watermark,text_5pyx6L-w5YuHLXpodXNodXlvbmc,color_ff0021,size_18,x_10,y_10)

- 刚开始 Monitor 中 Owner 为 null
- 当 Thread-2 执行 synchronized(obj)就会将 Monitor 的所有者 Owner 置为 Thread-2, Monitor 中只能有一个 Owner
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入
  EntryList BLOCKED
- Thread-2 执行完同步代码快的内容，然后唤醒 entryList 中等待的线程来竞争锁，竞争的时候是非公平的
- 图中WaitSet 中的 Thread-0, Thread-1 是之前获得过锁，但条件不满足进入 `waiting` 状态的线程


## wait(等待) notify(唤醒)

- `Owner(所有者)`线程发现条件不满足，调用 `wait` 方法，即可进入 `waitSet` 变为 `waiting` 状态
- `blocked`和`waiting`的线程都处于阻塞状态，不占用 CPU 时间片
- `blocked` 线程会在 `Owner(所有者)` 线程释放锁时唤醒
- `waiting` 线程会在 `Owner(所有者)` 线程调用 `notify` 或者 `notifyAll` 时唤醒，但唤醒后并不意味着立刻获得锁，仍需进入
   `等待队列` 重新竞争

### API 介绍

- `obj.wait()`让进入 object 监视器的线程到 waitSet 等待
- `obj.notify()` 在 object 上正在 waitSet 等待的线程中`挑一个唤醒`
- `obj.notifyAll()` 让 object 上正在 waitSet 等待的线程全部唤醒

**`它们都是线程之间进行协作的手段，都属于 Object 对象的方法，必须获得此对象的锁，才能调用这几个方法`**

```java
final static Object obj = new Object();
    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("执行....");
                try {
                    obj.wait(); // 让线程在obj上一直等待下去
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.debug("其它代码....");
            }
        }).start();
        new Thread(() -> {
            synchronized (obj) {
                log.debug("执行....");
                try {
                    obj.wait(); // 让线程在obj上一直等待下去
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.debug("其它代码....");
            }
        }).start();
        // 主线程两秒后执行
        sleep(2);
        log.debug("唤醒 obj 上其它线程");
        synchronized (obj) {
            obj.notify(); // 唤醒obj上一个线程
            // obj.notifyAll(); // 唤醒obj上所有等待线程
        }
    }
```

notify 的一种结果

```
20:00:53.096 [Thread-0] c.TestWaitNotify - 执行.... 
20:00:53.099 [Thread-1] c.TestWaitNotify - 执行.... 
20:00:55.096 [main] c.TestWaitNotify - 唤醒 obj 上其它线程
20:00:55.096 [Thread-0] c.TestWaitNotify - 其它代码....
```

notifyAll 的结果

```
19:58:15.457 [Thread-0] c.TestWaitNotify - 执行.... 
19:58:15.460 [Thread-1] c.TestWaitNotify - 执行.... 
19:58:17.456 [main] c.TestWaitNotify - 唤醒 obj 上其它线程
19:58:17.456 [Thread-1] c.TestWaitNotify - 其它代码.... 
19:58:17.456 [Thread-0] c.TestWaitNotify - 其它代码....
```

`wait()` 方法会释放对象的锁，进入 `WaitSet(等待队列)` 等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到 notify 为止

`wait(long n)` 有时限的等待, 到 n 毫秒后结束等待，或是被 notify

### sleep(long n) 和 wait(long n) 的区别

- sleep 是 Thread 的方法，而 wait 是 Object 的方法
- sleep 不需要强制和synchronized 配合使用， 但 wait 需要和 synchronized 一起用
- sleep 在睡眠的同时，不会释放对象锁。但是 wait 在等待的时候会释放对象锁
- 它们的状态都是 timed_waiting(有时间的等待)

```java
new Thread(() -> {
    synchronized (room) {
        log.debug("有烟没？[{}]", hasCigarette);
        while (!hasCigarette) {
            log.debug("没烟，先歇会！");
            try {
                room.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        log.debug("有烟没？[{}]", hasCigarette);
        if (hasCigarette) {
            log.debug("可以开始干活了");
        } else {
            log.debug("没干成活...");
        }
    }
}, "小南").start();

new Thread(() -> {
    synchronized (room) {
        Thread thread = Thread.currentThread();
        log.debug("外卖送到没？[{}]", hasTakeout);
        if (!hasTakeout) {
            log.debug("没外卖，先歇会！");
            try {
                room.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        log.debug("外卖送到没？[{}]", hasTakeout);
        if (hasTakeout) {
            log.debug("可以开始干活了");
        } else {
            log.debug("没干成活...");
        }
    }
}, "小女").start();
sleep(1);
new Thread(() -> {
    synchronized (room) {
        hasTakeout = true;
        log.debug("外卖到了噢！");
        room.notifyAll();
    }
}, "送外卖的").start();
```

```java
synchronized(lock) {
    while(条件不成立) {
      lock.wait();
    }
    // 干活
}
//另一个线程
synchronized(lock) {
    lock.notifyAll();
}
```











