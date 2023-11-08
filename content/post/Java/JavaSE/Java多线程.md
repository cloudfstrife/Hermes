---
title: "Java多线程"
date: 2019-03-04T15:27:01+08:00
categories:
- Java
tags:
- Java
keywords:
- Java
- 多线程
---

Java多线程

<!--more-->

现代操作系统是多任务操作系统。多线程是实现多任务的一种方式。

进程是指一个内存中运行的应用程序，每个进程都有自己独立的一块内存空间，一个进程中可以启动多个线程。比如在Windows系统中，一个运行的exe就是一个进程。

线程是指进程中的一个执行流程，一个进程中可以运行多个线程。比如java.exe进程中可以运行很多线程。线程总是属于某个进程，进程中的多个线程共享进程的内存。

> “同时”执行是人的感觉，在线程之间实际上轮换执行。

## Java中的线程

在java中要想实现多线程，有两种手段，一种是**继承Thread类**，另外一种是**实现Runable接口**。

### 继承Thread类

对于直接继承Thread的类来说，代码大致框架是：

```java
class 类名 extends Thread{

	属性1;
	属性2;
	...


	public void run(){
		
	}

	方法1;
	方法2;
	...
}
```

#### Demo：

ThreadTest.java

```java
package org.hibiscus.thread;

public class ThreadTest extends Thread {

	private String names;
	
	public String getNames() {
		return names;
	}

	public void setNames(String names) {
		this.names = names;
	}

	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			System.out.println(this.names+":\t"+i);
		}
	}
}
```

main.Java

```java
package org.hibiscus.main;

public class Main {

	public static void main(String[] args) {
		ThreadTest testA = new ThreadTest();
		testA.setNames("A");
		ThreadTest testB = new ThreadTest();
		testB.setNames("B");
		testA.start();
		testB.start();
	}
	
}
```

执行Main，会看到A与B的交替打印

> 有时因为系统分配CPU时间的问题。可能出现先打印A再打印B，或者先打印B后打印A的情况，可以将循环次数改大一点，或者多运行几次可以看到效果。

这里使用的是start()方法而不是run()，因为多线程需要系统支持，所以start()方法调用系统底层函数来运行run()方法以实现多线程。如果start方法重复调用，会出现java.lang.IllegalThreadStateException异常。

### 实现Runnable接口

对于实现Runnable接口来说，代码大致框架是：

```java
class 类名 implements Runnable{

	属性1;
	属性2;
	...
	public void run(){

	}

	方法1;
	方法2;
	...
}
```

#### Demo

RunnableTest.java

```java
package org.hibiscus.thread;

public class RunnableTest implements Runnable {

	private String names;
	
	public String getNames() {
		return names;
	}

	public void setNames(String names) {
		this.names = names;
	}

	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			System.out.println(this.names+":\t"+i);
		}
	}

}
```

Main.java

```java
package org.hibiscus.main;

import org.hibiscus.thread.RunnableTest;

public class Main {

	public static void main(String[] args) {
		RunnableTest testA = new RunnableTest();
		testA.setNames("A");
		RunnableTest testB = new RunnableTest();
		testB.setNames("B");
		Thread threadA = new Thread(testA);
		Thread threadB = new Thread(testB);
		threadA.start();
		threadB.start();
	}
}
```

> 关于选择继承Thread还是实现Runnable接口？
> 
> &emsp;&emsp;如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

#### Demo：

```java
class Saler implements Runnable{
 
	private int ticket = 5;  //5张票
 
	public void run() {
		for (int i=0; i<=20; i++) {
			if (this.ticket > 0) {
				System.out.println(Thread.currentThread().getName()+ "正在卖票"+this.ticket--);
			}
		}
	}
}

public class Main {
	 
	public static void main(String [] args) {
		Saler my = new Saler();
		new Thread(my, "1号窗口").start();
		new Thread(my, "2号窗口").start();
		new Thread(my, "3号窗口").start();
	}
}
```

> 实现Runnable接口比继承Thread类所具有的优势：
> 
> 1. 适合多个相同的程序代码的线程去处理同一个资源
> 
> 1. 可以避免java中的单继承的限制
> 
> 1. 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立。


注意：
**main方法其实也是一个线程。在java中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。**
 
在java中，每次程序运行至少启动**2个线程**。一个是main线程，一个是垃圾收集线程。

因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM就是在操作系统中启动了一个进程。

## 线程的生命周期

线程的生命周期是指一个线程从创建到消亡的过程。如下图示：

![](/images/Java/multi_thread_01.png)

线程的生命周期可以分为四个状态:

**创建状态**

当用new操作符创建一个新的线程对象时，该线程处于创建状态。	

**可运行状态**

执行线程的start()方法将为线程分配必须的系统资源，安排其运行，并调用线程体——run()方法，这样就使得该线程处于可运行状态（Runnable）。

这一状态并不是运行中状态（Running），因为线程也许实际上并未真正运行。

**不可运行状态**

当发生下列事件时，处于运行状态的线程会转入到不可运行状态：

1. 调用了sleep()方法；

2. 线程调用wait()方法等待特定条件的满足；

3. 线程输入/输出阻塞。

**返回可运行状态**

1. 处于睡眠状态的线程在指定的时间过去后；

1. 如果线程在等待某一条件，另一个对象必须通过notify()或notifyAll()方法通知等待线程条件的改变；

2. 如果线程是因为输入输出阻塞，等待输入输出完成。 

* 消亡状态

&emsp;&emsp;当线程的run()方法执行结束后，该线程自然消亡。

## Thread的常用方法

* isAlive()

```java
public final boolean isAlive()
```

> 测试线程是否处于活动状态。如果线程已经启动且尚未终止，则为活动状态。 

* isDaemon()

```java
public final boolean isDaemon()
```

> 测试该线程是否为守护线程。 

* isInterrupted()

```java
public boolean isInterrupted()
```

> 测试线程是否已经中断。线程的中断状态 不受该方法的影响。 
>
> 线程中断被忽略，因为在中断时不处于活动状态的线程将由此返回 false 的方法反映出来。 

* join()

```java
public final void join() throws InterruptedException
public final void join(long millis) throws InterruptedException
public final void join(long millis,int nanos) throws InterruptedException
```

> 等待该线程终止。带参数的方法等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。

* sleep()

```java
public static void sleep(long millis) throws InterruptedException
public static void sleep(long millis,int nanos) throws InterruptedException
```

> 在指定的毫秒数加指定的纳秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。该线程不丢失任何监视器的所属权。 

* interrupt()

```java
public void interrupt()
```

> 中断线程。
> 
> 如果当前线程没有中断它自己（这在任何情况下都是允许的），则该线程的 checkAccess 方法就会被调用，这可能抛出 SecurityException。 

> 如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，则其中断状态将被清除，它还将收到一个 InterruptedException。 

> 如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到一个 ClosedByInterruptException。 

> 如果该线程在一个 Selector 中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。 

> 如果以前的条件都没有保存，则该线程的中断状态将被设置。

> 中断一个不处于活动状态的线程不需要任何作用。 

* setPriority()

```java
public final void setPriority(int newPriority)
```

> 更改线程的优先级。 

> 不要误以为优先级越高就先执行。谁先执行还是取决于谁先去的CPU的资源。
> 
> 线程的默认优先级是5
> 
> Thread提供了三个常量
> 
> 1. **Thread\.MAX_PRIORITY** &emsp;&emsp;&emsp;&emsp;线程可以具有的最高优先级。
>  
> 1. **Thread\.MIN_PRIORITY** &emsp;&emsp;&emsp;&emsp;线程可以具有的最低优先级。
> 
> 1. **Thread\.NORM_PRIORITY**  &emsp;&emsp;&emsp;分配给线程的默认优先级。 

* yield()

```java
public static void yield()
```

> 暂停当前正在执行的线程对象，并执行其他线程。 
