---
title: '[Java多线程]基础线程机制'
date: 2019-07-10 22:01:13
tags:
- Java基础
- 笔记
- 多线程
---

## 一、线程状态

线程可以有如下六种状态：

- New（新创建）
- Runnable（可运行）
- Blocked（被阻塞）
- Waiting（等待）
- Timed Waiting（计时等待）
- Terminated（被终止）

<!--more-->

可以使用getState方法获取一个线程的状态

<img src="states.png">



参考自[CyC2018](<https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%B9%B6%E5%8F%91.md#%E4%B8%80%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2>)

### 新建（New）

创建后尚未启动。

### 可运行（Runnable）

可能正在运行，也可能正在等待 CPU 时间片。

包含了操作系统线程状态中的 Running 和 Ready。

### 阻塞（Blocked）

等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

### 无限期等待（Waiting）

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

### 限期等待（Timed Waiting）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

### 死亡（Terminated）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

## 二、使用线程

三种方法：

- 实现Runnable接口
- 继承Thread类
- 实现Callable接口

### 实现接口VS继承Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## 三、Java线程机制

### Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

### Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

守护线程永远不应该访问固有资源，如文件、数据库等，因为他可能在任意时间中断。

## 四、中断线程

- run方法执行方法体中最后一条语句，并经由return语句返回
- 出现了方法中未捕获的异常

Java没有可以强制中断线程的方法，只提供了一种协作机制：

调用interrupt()方法将会使中断标志置位（设为true），但是并不会中断线程，需要程序员自己写代码判断中断标志是否置位，然后执行相关操作

### 阻塞状态

一个任务进入阻塞状态可能有如下原因：

- 调用sleep()方法使任务进入休眠状态
- 调用wait()方法使线程挂起
- 任务在等待某个输入输出完成
- 任务试图调用其同步控制方法，但是对象锁不可用

### void interrupt()

向线程发送中断请求，中断标志设为true，如果线程被阻塞，抛出InterruptException。但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

### static boolean interrupted()

测试正在执行这一命令的线程是否被中断，并将当前线程的中断标志重置为false

### boolean isInterrupted()

测试当前线程是否被中断，不会改变线程的中断状态

```java
class MyThread2 extends Thread{
    @Override
    public void run() {
        while(!interrupted()){
            //...
        }
        System.out.println("Thread interrupted");
    }
}
public class InterruptTest {
    public static void main(String[] args) {
        MyThread2 thread1 = new MyThread2();
        thread1.start();
        thread1.interrupt();
        System.out.println("Main run");
    }
}


Main run
Thread interrupted
```

如果循环调用sleep，则无需检测中断标志

```java
class MtThread1 extends Thread{
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            System.out.println("Thread finished");
        }
    }
}
public class InterruptTest {
    public static void main(String[] args) {
        MtThread1 thread1 = new MtThread1();
        thread1.start();
        thread1.interrupt();
        System.out.println("Main run");
    }
}


Main run
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at com.wdw.thread.MtThread1.run(InterruptTest.java:7)
Thread finished
```



