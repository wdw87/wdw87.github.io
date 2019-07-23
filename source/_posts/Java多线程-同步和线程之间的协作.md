---
title: '[Java多线程]同步和线程之间的协作'
date: 2019-07-23 17:51:25
tags:
- Java基础
- 笔记
- 多线程
---

# 五、同步

## 竞态条件

当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。

一个存在竞态条件的例子：

<!--more-->

```java
class Counter {
    protected long count;
    public Counter(){};
    public Counter(long count){
        this.count = count;
    }
    public void add() {
        count++;
    }
}

public class SynchronizeTest {
    public static void main(String[] args) {
        Counter counter = new Counter(0);

        for(int i = 0; i < 100; i++){
            new Thread(()->{
                //加入延时，模拟并发条件
                try{
                    Thread.sleep(100);
                    counter.add();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }).start();
        }
        //等待100条线程都执行完毕
        try{
            Thread.sleep(100);
            System.out.println(counter.count);
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }
}
```

在上述代码中，理应输出结果100，但是执行后每次结果都不同，且都不是100，原因在于100条线程同时访问Counter对象，JVM对于Counter对象的add()方法是按照如下顺序执行的：

```java
从内存获取 this.count 的值放到寄存器
将寄存器中的值增加value
将寄存器中的值写回内存
```

如果此时其中两条线程同时执行，那么执行顺序极有可能是下面的情况：

```java
	this.count = 0;
   A:	读取 this.count 到一个寄存器 (0)
   B:	读取 this.count 到一个寄存器 (0)
   B: 	将寄存器的值加1
   B:	回写寄存器值(1)到内存. this.count 现在等于 1
   A:	将寄存器的值加1
   A:	回写寄存器值(1)到内存. this.count 现在等于 1
```

当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。上例中add()方法就是一个临界区,它会产生竞态条件。在临界区中使用适当的同步就可以避免竞态条件。

## 同步

Java提供了两种机制，防止代码块受并发访问的干扰，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

基本上所有的并发模式在解决线程冲突问题的时候，都是采用**序列化访问共享资源**的方案。通常在代码前加上一条锁语句实现。锁语句产生了一种互相排斥的效果，所以这种机制常常称为**互斥量**。

> Brian的同步规则
>
> 如果你正在写一个变量，它可能接下来被另外一个线程读取，或者正在读取一个上一次已经被另外一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须使用相同的监视器锁同步。

### ReentrantLock 

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

Lock对象必须显示的创建、锁定、释放。

基本结构如下：

```java
myLock.lock();
try{
    //...
}finally{
    myLock.unlock();// 确保释放锁，从而避免发生死锁。
}
```

> 吧解锁放在finally中至关重要，如果代码在临界区抛出异常，所必须释放，否则其他线程将永久阻塞。

抢票Demo中的应用：

```jaVA
class Station implements Runnable{
    private int tickets = 100;
    private Lock myLock = new ReentrantLock();
    @Override
    public void run() {
        while(true){
            try {
                myLock.lock();
                Thread.sleep(20);
                if(tickets > 0){
                    tickets--;
                    System.out.println(Thread.currentThread().getName() + "->" + tickets);
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }finally{
                myLock.unlock();// 确保释放锁，从而避免发生死锁。
            }
        }
    }
}
public class TicketExample {
    public static void main(String[] args) {
        Station station = new Station();
        new Thread(station).start();
        new Thread(station).start();
        new Thread(station).start();
        System.out.println("Main run");
    }
}
```

### synchronized

有如下四种用法：

- 同步一个的代码块
- 同步一个方法
- 同步一个类
- 同步一个静态方法，以Class对象作为锁

#### 1. 同步一个代码块

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

使用同步代码块实现的模拟抢票

```java
class Station implements Runnable {
    private int tickets = 100;
    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                try {
                    Thread.sleep(10);
                    if (tickets > 0) {
                        tickets--;
                        System.out.println(Thread.currentThread().getName() + "->" + tickets);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

2. 同步方法

```java
public synchronized void func () {
    // ...
}
```

3. 同步一个类

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
class Station implements Runnable {
    private int tickets = 10;
    @Override
    public void run() {
        while (true) {
            synchronized (Station.class) {
                try {
                    Thread.sleep(10);
                    if (tickets > 0) {
                        tickets--;
                        System.out.println(Thread.currentThread().getName() + "->" + tickets);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
public class TicketExample {
    public static void main(String[] args) {
        Station station = new Station();
        Station station1 = new Station();
        //注意这里访问的是同一个类的不同实现
        new Thread(station).start();
        new Thread(station1).start();
        System.out.println("Main run");
    }
}


输出也发生了同步：
Main run
Thread-0->9
Thread-0->8
Thread-0->7
Thread-0->6
Thread-0->5
Thread-0->4
Thread-0->3
Thread-0->2
Thread-0->1
Thread-0->0
Thread-1->9
Thread-1->8
Thread-1->7
Thread-1->6
Thread-1->5
Thread-1->4
Thread-1->3
Thread-1->2
Thread-1->1
Thread-1->0
```

4. 同步一个静态方法

```java
public synchronized static void fun() {
    // ...
}
```

作用于整个类。

# 六、线程之间的协作

## join()

一个线程可以在其他线程上调用join()方法，其效果是等待一段时间知道第二个线程结束才继续执行。

```java
class A extends  Thread{
    @Override
    public void run() {
        System.out.println("A running");
    }
}
class B extends Thread{
    private A a;
    public B(A a){
        this.a = a;
    }
    @Override
    public void run() {
        try {
            a.join();
            System.out.println("B running");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
public class JoinTest {
    public static void main(String[] args) {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}


A running
B running
```

## wait()  notify() 和 notifyAll()

调用wait()使线程进入无限等待状态，只有在notify() 和 notifyAll()发生时，这个线程才会被唤醒去检查所发生的变化。

wait()  notify() 和 notifyAll()使Object类的final方法

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。且调用这些方法的线程必须已经拥有对象的锁。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。



**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

Java编程思想中的经典Demo，顾客和服务员的消费者生产者例子。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

class Meal{
    private final int orderNum;
    public Meal(int orderNum){this.orderNum = orderNum;}
    public String toString(){ return "Meal " + orderNum;}
}
class WaitPerson implements Runnable{
    private Restaurant restaurant;
    public WaitPerson(Restaurant r) { restaurant = r;}
    @Override
    public void run() {
        try {
            while(!Thread.interrupted()){
                synchronized (this){  //注意，这里的this指向waitPerson，所以只有waitPerson.notifyAll()才能唤醒这个等待
                    while(restaurant.meal == null){
                        wait();
                    }
                }
                System.out.println("wait person got " + restaurant.meal);
                synchronized (restaurant.chef){
                    restaurant.meal = null;
                    restaurant.chef.notifyAll();
                }
            }
        } catch (InterruptedException e) {
            System.out.println("WaitPerson interrupted");
        }
    }
}
class Chef implements Runnable{
    private Restaurant restaurant;
    private int count = 0;
    public Chef(Restaurant r) {restaurant = r;}
    @Override
    public void run() {
        try {
            while (!Thread.interrupted()){
                synchronized (this) {
                    while (restaurant.meal != null)
                        wait();
                }
                if(++count == 10){
                    System.out.println("Out of food,closing");
                    //向所有ExecutorService启动的任务发送interrupt()
                    restaurant.exec.shutdownNow();
                    //return;  //如果不将return注释，则不会抛出异常，直接返回
                }
                System.out.println("Order up !!");
                synchronized (restaurant.waitPerson){
                    restaurant.meal = new Meal(count);
                    restaurant.waitPerson.notifyAll();
                }
                TimeUnit.MILLISECONDS.sleep(100);
            }
        } catch (InterruptedException e) {
            System.out.println("Chef interrupted");
        }
    }
}
public class Restaurant {
    Meal meal;
    ExecutorService exec = Executors.newCachedThreadPool();
    WaitPerson waitPerson = new WaitPerson(this);
    Chef chef = new Chef(this);
    public Restaurant(){
        exec.execute(chef);
        exec.execute(waitPerson);
    }

    public static void main(String[] args) {
        new Restaurant();
    }
}


Order up !!
wait person got Meal 1
Order up !!
wait person got Meal 2
Order up !!
wait person got Meal 3
Order up !!
wait person got Meal 4
Order up !!
wait person got Meal 5
Order up !!
wait person got Meal 6
Order up !!
wait person got Meal 7
Order up !!
wait person got Meal 8
Order up !!
wait person got Meal 9
Out of food,closing
WaitPerson interrupted
```

上述例子中有许多值得注意的点：

- 调用notifyAll()唤醒waitPerson时，必须先捕获waitPerson上的锁。同时，waitPerson.run() 中的wait()会自动的释放这个锁。
- 在调用restaurant.exec.shutdownNow()后，向所有ExecutorService启动的任务发送interrupt()，WaitPerson任务由于处于阻塞状态（调用了wait()），抛出InterruptedException异常，并在异常捕获后结束。但是Chef任务直到试图调用sleep() 方法时，试图进入一个阻塞操作，才抛出InterruptedException异常。
- 使用**阻塞队列**实现生产者消费者模式是一种更明智的选择

## await() signal() 和 signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

```java
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

通常，wait() 和await() 的调用都应该在循环体中：

```java
while(!(ok to process)){
    wait();
    //condition.await();
}

```

目的是防止信号丢失和假唤醒，以免造成死锁。

