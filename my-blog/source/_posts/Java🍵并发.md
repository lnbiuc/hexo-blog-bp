---
title: Java🍵并发
date: 2022/10/10 21:33:37
updated: 2023/3/23 23:02:32
categories:
- Java
tags:
- Java
- Concurrency
---

# 线程和进程

## 进程

在Java中：进程是程序的一次执行过程，是系统运行程序的基本单位。

在操作系统中：

1. 进程是程序的一次执行。
2. 进程是一个程序及数据在处理机上顺序执行时所发生的活动。
3. 进程是具有独立功能的程序在一个数据集合上运行的过程，是系统资源分配和调度的一个独立单位。

## 线程

线程与进程类似，线程是一个比进程跟小的执行单位。一个进程在其执行过程中可以创建多个线程。

在Java中：多个线程共享进程的堆和方法去资源，但每个线程都有自己的**程序计数器**、**虚拟机栈**、**本地方法栈**。

## 相关问题

### 1. 线程与进程的关系

- 一个进程中可以有多个线程，线程是进程划分成的更小的运行单位。
    - 多个线程共享进程的**堆**和**方法区**（在JDK1.8之后方法区教元空间）。
    - 程序计数器、虚拟机栈、本地方法栈每个线程独享。
- 各个进程之间是独立的。

### 2. 程序计数器为什么是线程私有的

- 字节码解释器通过改变程序计数器指向的位置一次读取执行命令，从而实现代码流程的控制。
- 在多线程的情况下，程序计数器用于记录当前线程的执行位置，从而当前线程停止或切换的时候能够知道该线程中程序执行到什么位置。

所以程序计数器是线程私有的，为了记录程序执行的位置。

### 3. 虚拟机栈和本地方法栈为什么是私有的

- **虚拟机栈**：每个Java方法执行时会创建一个栈帧用于保存局部变量表、操作数栈、常量池引用等数据。当一个方法被调用到执行完成的过程，对应一个栈帧在Java虚拟机栈中入栈到出栈的过程。
- **本地方法栈**：和虚拟机栈的功能相似。区别在于：虚拟机栈为虚拟机执行Java方法（字节码）服务；本地方法栈为虚拟机执行Native方法服务。

### 4. 堆区和方法区为什么是共享的

- **堆**是进程中最大的一块内存，主要用于存放新创建的对象。
- **方法区**存放已经被加载的类的信息、常量、静态变量、即时编译器编译后的代码等信息。

# 线程的生命周期

- New：初始状态，线程被创建还没有没调用。
- ~~Ready：就绪状态，线程以具备了运行的条件、资源。~~
- Runnable：运行状态，线程被调用。
- Blocked：阻塞状态，需要等待资源或锁释放才能继续执行。
- Waiting：等待状态，该线程需要等待其他线程先执行，会一直等待。
- Time_Waiting：超时等待状态，可以在指定时间后自行返回。
- Terminated：终止状态，表示该线程已经运行完毕。

# 上下文切换

上下文：进程运行的条件和状态，如程序计时器、栈信息等。

线程在运行过程中出现一些原因，使线程从CPU占用状态退出：

- 主动让出CPU，如调用sleep、wait方法
- 时间片用完
- 调用了IO请求
- 线程终止或结束运行

线程状态发生变化，表明需要记录线程此时的上下文，等待线程唤醒重新占用CPU时回复状态，或线程执行完毕，加载下个线程，就要发生上下文切换。

# 死锁

## 什么是死锁

线程死锁是指：多个线程同时被阻塞，如果某一个资源一直得不到释放，多个线程将被无限阻塞，程序不能正常运行。

## 产生条件

- 互斥条件：该资源任意时刻只能由一个线程占用。
- 请求与保持条件：一个线程因为请求的资源未被释放而阻塞，同时对自己已获得的资源不释放。
- 不剥夺条件：线程已经获取的资源，不使用完毕不释放，同时不被其他线程剥夺。
- 循环等待条件：多个线程之间形成头尾相接的循环等待资源关系。

## 预防死锁

- 破坏请求与保持条件：一次性申请所有资源。
- 破坏不剥夺条件：如果申请其他资源一直申请不到，可以先释放自身占有的资源。
- 破坏循环等待条件：按序申请资源。

## 避免死锁

资源分配时，借助算法（银行家算法）对资源分配进行计算评估，是线程进入安全状态。

# sleep() & wait()

sleep和wait都可用于暂停线程执行。

- sleep没有释放锁，wait释放了锁（为了能被notify）。
- wait常用于线程之间交互、通讯，sleep用于暂停执行。
- wait调用之后线程不会自动唤醒，需要别的线程执行notify()方法才能唤醒。
- sleep可以设置时间，到时自动唤醒。
- sleep是thread类的静态本地方法，wait是Object类的本地方法。

# start() & run()

## start

创建一个线程，线程此时是创建状态，调用start方法，jvm会启动一个线程并使该线程进入就绪态，等待CPU分配时间片之后线程开始运行。

start会执行线程的准备工作，之后执行run方法里的内容。

## run

直接调用run方法，会把run当成一个main函数下的普通方法执行，不会在某个线程中执行。

# 线程安全

## 线程不安全示例

```java
public class UnSafeThreadExample
{
    public static void main(String[] args) throws InterruptedException
    {
        final int THREAD_SIZE = 1000;
        FuncAdd funcAdd = new FuncAdd();
        final CountDownLatch countDownLatch = new CountDownLatch(THREAD_SIZE);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < THREAD_SIZE; i++)
        {
            executorService.execute(() -> {
                funcAdd.add();
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        System.out.println("funcAdd.getI() = " + funcAdd.getI());
    }
}

class FuncAdd
{
    private int i = 0;
    
    void add()
    {
        i++;
    }
    
    int getI()
    {
        return i;
    }
}
```

## 出现原因

- 可见性：一个线程对共享变量的修改，另一个线程可以马上读取到。
- 原子性：cpu不能保证多个线程时，每个线程每次执行一步，有可能某个线程执行了两步。
- 有序性：执行顺序与代码顺序不同。

## 线程安全的定义

一个类可以被多个线程安全调用

# 线程创建方式（4种）

1. Runnable
    1. 实现Runnable接口，实现run方法
    2. 创建Runnable实现类对象，传入Thread构造函数
    3. 调用start方法启动线程
2. Callable
    1. 实现Callable接口，实现call方法
    2. 通过FutureTask的构造方法，传入Callable实现类对象
    3. 创建Thread对象，传入futureTask
    4. 调用start方法启动线程，也可调用futureTask.get获取执行结果
3. Thread
    1. 继承Thread类，重写run方法
    2. 创建Thread子类对象，调用start方法
4. 使用线程池创建
    1. 实现Runnable，实现run方法
    2. 创建线程池
    3. 通过ExecutorService的executor方法传入Runnable实现类对象

> Runnable接口不会返回结果或抛出异常，Callable接口可以（Callable实现类中call方法有返回值）

实现Runnable和Callable接口的类，相当于创建了在线程中运行的任务，最后要把这项任务交给Thread调用，使用start()方法执行。

优先使用实现接口的方式：继承Thread类之后就不能继承其他类。

## Runnable

```java
public class CreateThread
{
    public static void main(String[] args)
    {
        // runnable
        UseRunnable runnable = new UseRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
    }
}

class UseRunnable implements Runnable
{
    
    @Override
    public void run()
    {
    	
    }
}
```

## Callable

```java
public class CreateThread
{
    public static void main(String[] args)
    {
        // callable
        UseCallable useCallable = new UseCallable();
        FutureTask<Object> task = new FutureTask<>(useCallable);
        Thread thread1 = new Thread(task);
        thread1.start();
    }
}


class UseCallable implements Callable<Object>
{
    @Override
    public Object call() throws Exception
    {
        return null;
    }
}
```

## Thread

```java
public class CreateThread
{
    public static void main(String[] args)
    {
        // thread
        UseThread thread2 = new UseThread();
        thread2.start();
    }
}

class UseThread extends Thread
{
    public void run()
    {
    
    }
}
```

## 线程池

创建线程池的7中方式

```java
// 创建线程池
ExecutorService executorService = Executors.newCachedThreadPool();
	executorService.execute(
        () -> {
			func()
		}
	);
```

# 线程相关方法

## Executor

Executor管理多个异步任务的执行，无需显示控制线程生命周期。Executor管理的多个任务执行互不干扰，不需要进行同步操作。

Executor有三个重要方法：创建线程池的方式：

- CachedThreadPool：一次任务创建一个线程。
- FixedThreadPool：所有任务只能使用固定大小的线程。
- SingleThreadExecutor：固定大小为1的线程。

```java
final CountDownLatch countDownLatch = new CountDownLatch(THREAD_SIZE);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < THREAD_SIZE; i++)
        {
            executorService.execute(() -> {
                funcAdd.add();
                countDownLatch.countDown();
            });
        }
```

## Daemon

守护线程，程序运行时在后台提供服务的线程。

所有非守护线程结束后，守护线程也结束。可以使用`setDaemon()`将一个线程设置为守护线程

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

## yield

表示跟当前线程具有相同优先级的线程可以先运行

# 线程中断

- 线程执行完自动中断
- 线程执行过程中发生错误中断
- 主动调用`interrupt()`中断线程
- 调用`Executor.shutdown()`会等待所有线程执行完之后中断，调用`Executor.shutdownNow()`会立即对所有线程执行`interrupt()`方法

# 线程互斥同步

使用synchoronized和ReentrantLock这两种锁来使多个线程对共享资源的互斥访问。

## synchoronized（同步锁）

基于JVM实现的锁

- 是排他锁，只能被一个线程获取。
- 如果锁住的是方法或者代码块，每个对象都有一把锁，不同对象之间互不影响。
- 如果锁住的是类或者静态方法，所有该类的对象共用一把锁。
- synchronized会自动释放，在方法执行完或抛出异常都会释放锁。

### 用法

#### 代码块

使用synchronized同步代码块，只能用于同一个对象（一个方法执行两次），如果调用的是两个对象（使用new创建了两个对象），则不能同步。

当一个线程进入同步代码块时，另一个线程必须等待。

```java
public class UseSynchronized
{
    /*
    synchronized同步代码块
     */
    public void func1()
    {
        synchronized (this)
        {
            // 代码块
            int[] arr = new int[]{1,3,4,5,6};
            for (int i : arr)
            {
                System.out.println("i = " + i);
            }
        }
    }
}

class AppUseSynchronized
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newCachedThreadPool();
        UseSynchronized s1 = new UseSynchronized();
        executorService.execute(s1::func1);
        executorService.execute(s1::func1);
    }
}
```

输出结果：

```
 1  3  4  5  6  1  3  4  5  6 
```

第一个线程进入同步代码块，必须等待代码块中方法执行完之后，才允许另一个线程进入。

如果用于多个对象，线程不能同步

```java
class AppUseSynchronized
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newCachedThreadPool();
        UseSynchronized s1 = new UseSynchronized();
        UseSynchronized s2 = new UseSynchronized();
        executorService.execute(s1::func1);
        executorService.execute(s2::func1);
    }
}
```

输出结果：

```
 0  1  2  3  4  5  6  7  0  1  2  3  4  5  6  7  8  9  8  9 
```

#### 方法

与代码块相同，只能作用于一个对象

```java
public class UseSynchronized
{
 	// synchronized同步方法   
    public synchronized void func2()
    {
        for (int i = 0; i < 10; i++)
        {
            System.out.print(" " + i + " ");
        }
    }
}
```

#### 类

多个线程调用同一个类的多个对象时，也会同步

```java
/*
同步一个类
 */
class SynchronizedClass
{
    public void func1()
    {
        synchronized (SynchronizedClass.class)
        {
            for (int i = 0; i < 10; i++)
            {
                System.out.print(" " + i + " ");
            }
        }
    }
}


class AppUseSynchronized
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newCachedThreadPool();
        SynchronizedClass s1 = new SynchronizedClass();
        SynchronizedClass s2 = new SynchronizedClass();
        executorService.execute(s1::func1);
        executorService.execute(s2::func1);
    }
}
```

输出结果：

```
 0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5  6  7  8  9 
```

#### 静态方法

和类一样。

```java
public synchronized static void fun() 
{
    for (int i = 0; i < 10; i++)
		{
			System.out.print(" " + i + " ");
		}
}
```

### Synchronized与Lock

两者都是可重入锁

Synchronized：

- 效率低：锁不容易释放，获取锁不能设置超时时间。
- 不知加锁是否成功
- 依赖于JVM

Lock：

- 可以尝试获取锁，并设置超时时间
- 可以手动释放锁
- 依赖于Java API

### 锁优化

- 锁粗化：将多个连续的锁扩展成一个范围更大的锁
- 锁消除：通过逃逸分析消除不会产生竞争的代码上的锁
- 适应性自旋锁：获取轻量级锁失败过程中，执行CAS失败时，多次尝试获取锁，一定次数后阻塞

### 锁升级

无锁 ->  偏向锁 -> 轻量级锁 -> 重量级锁

在JDK1.5之前，synchoronized是重量级锁。在JDK1.6，synchoronized获取了优化，有了从无锁到重量级锁的升级

#### 用户态和内核态

内核态：控制计算机硬件资源，协调CPU资源，分配内存资源等

用户态：提供程序运行的空间，提供CPU、内存、IO资源的访问接口

synchoronized依赖操作系统实现，在使用synchoronized同步锁时，需要由用户态切换到内核态

synchoronized底层monitorenter和monitorexit依赖操作系统的Mutex Lock实现，使用Mutex Lock需要当前线程挂起并从用户态切换到内核态

#### 无锁 -> 偏向锁

1. 访问对象头中Mark Word中偏向锁标志是否为0，锁标志是否为01，确认可进入偏向状态
2. 判断线程ID是否为当前线程，如果相同直接获得锁
3. 如果线程ID不相同，通过CAS竞争锁，竞争成功将线程ID修改为当前线程ID，获取锁
4. 如果CAS获取偏向锁失败，偏向锁升级为轻量级锁

#### 偏向锁 -> 轻量级锁

当有另一个线程竞争获取这个锁时，该锁已经时偏向锁，且对象头中线程ID不相同，通过CAS尝试加锁（自旋）如果成功不进行锁升级，如果失败则升级为重量级锁

## ReentrantLock（可重入锁）

基于JUC（ java.util.concurrent）实现的锁

只能作用于同一个对象

```java
public class UseReentrantLock
{
    private final Lock lock = new ReentrantLock();
    
    public void func1()
    {
        lock.lock();
        for (int i = 0; i < 10; i++)
        {
            System.out.print(" " + i + " ");
        }
        lock.unlock();
    }
}

class AppUseReentrantLock
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newCachedThreadPool();
        UseReentrantLock r1 = new UseReentrantLock();
        UseReentrantLock r2 = new UseReentrantLock();
        executorService.execute(r1::func1);
        executorService.execute(r1::func1);
    }
}
```

输出结果：

```
 0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5  6  7  8  9 
```

### synchoronized和ReentrantLock

- 实现方式：synchronized基于JVM实现，ReentrantLock基于JUC实现。
- 等待可中断：synchronized不可中断，ReentrantLock可中断。
- 公平锁（多个线程在等待同一个锁时，必须按照申请锁的时间顺序来一次获得锁）：两者都是非公平锁。

优先使用synchronized，synchronized可以自动释放锁，ReentrantLock不会自动释放

## volatile

volatile关键字表示一个变量是共享且不稳定的，每次使用该变量都需要从主存中读取。volatile能保证数据的可见性。

正常情况下，线程或操作本地内存中的共享变量副本，这会导致不可见问题（一个线程修改了变量，而另一个线程不能及时知道）。使用volatile可以强制每次读取变量都从主存中读取，从而避免了可见性问题。

### volatile与Synchronized比较

- volatile属于轻量级锁，volatile性能优于Synchronized。
- volatile只能用于变量，Synchronized可以用于方法、静态代码块、静态方法、类。
- volatile能保证数据可见性，但不能保证原子性，Synchronized都能保证。
- volatile主要解决变量在多个线程之间的可见性问题，Synchronized解决多个线程访问资源的同步问题。

# 线程之间合作（调度）

## join()

在当前线程中调用另一个线程的join方法，会将当前线程挂起，知道另一个线程结束，才继续执行当前线程。

```java
public class UseJoin
{
    class A extends Thread
    {
        @Override
        public void run()
        {
            System.out.println("A.run");
        }
    }
    
    class B extends Thread
    {
        private A a;
        
        B(A a)
        {
            this.a = a;
        }
        
        @Override
        public void run()
        {
            try
            {
                // b线程joina线程，必须等待a线程执行完，才能执行b线程
                a.join();
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }
            System.out.println("B.run");
        }
    }
    public void useJoin()
    {
        A a = new A();
        B b = new B(a);
        // 先启动b线程
        b.start();
        a.start();
    }
}

class AppUseJoin
{
    public static void main(String[] args)
    {
        UseJoin join = new UseJoin();
        join.useJoin();
    }
}
```

输出结果：

```
A.run
B.run
```

## wait() notify() notifyAll()

当线程需要某个条件时，进入wait状态，当条件满足时，有其他线程调用notify方法唤醒wait的线程

wait只能在同步方法或同步控制块中使用，否则会抛出异常IllegalMonitorStateExeception（非法监控状态执行）

线程wait期间，线程锁（synchoronized）释放。如果不释放线程锁，其他进程无法进入对象同步方法或控制块中，也就无法执行notify方法唤醒线程，造成死锁。

```java
public class WaitAndNotify
{
    public synchronized void before()
    {
        notifyAll();
        System.out.println("notify");
    }
    
    public synchronized void after()
    {
        try
        {
            wait();
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        System.out.println("wait");
    }
}

class AppWaitAndNotify
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newCachedThreadPool();
        WaitAndNotify w1 = new WaitAndNotify();
        executorService.execute(w1::after);
        executorService.execute(w1::before);
    }
}
```

## await() signal() signalAll()

await相比于wait可以设置等待条件

# 线程池

## 线程池优势

- 降低资源消耗：复用已经创建的线程，减少线程创建销毁的消耗。
- 提高响应速度：任务到达可以直接执行，不用在创建线程。
- 提高线程的可管理性：线程资源由线程池统一分配，调度，监控。

## 创建线程池

使用`ThreadPoolExecutor`构造函数创建线程池，不建议使用`Executor`直接创。

- 1. 通过ThreadPoolExecutor构造方法创建
- 通过ExecutorService下的工具类Executors创建：当任务被提交时，如果有空闲线程则立即执行，没有则进入等待队列

    - 2. CachedThreadPool：有空线程优先复用空线程，没有就创建新线程
    - 3. FixedThreadPool：所有任务只能使用固定大小的线程
    - 4. SingleThreadExecutor：固定大小为1的线程。
    - 5. ScheduledThreadPool：可以延时执行任务的线程池
    - 6. SingleThreadScheduledExecutor：单线程可以执行延时任务的线程池
    - 7. WorkStealingPool：抢占式执行任务的线程池

### 1. ThreadPoolExecutor

```java
public static void myThreadPoolExecutor() {
    // 创建线程池
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(5, 10, 100, TimeUnit.SECONDS, new LinkedBlockingQueue<>(10));
    // 执行任务
    for (int i = 0; i < 10; i++) {
        final int index = i;
        threadPool.execute(() -> {
            System.out.println(index + " 被执行,线程名:" + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```

### 2. FixedThreadPool

```java
public static void fixedThreadPool() {
    // 创建 2 个数据级的线程池
    ExecutorService threadPool = Executors.newFixedThreadPool(2);

    // 创建任务
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("任务被执行,线程:" + Thread.currentThread().getName());
        }
    };

    // 线程池执行任务(一次添加 4 个任务)
    // 执行任务的方法有两种:submit 和 execute
    threadPool.submit(runnable);  // 执行方式 1:submit
    threadPool.execute(runnable); // 执行方式 2:execute
    threadPool.execute(runnable);
    threadPool.execute(runnable);
}
```

### 3. CachedThreadPool

```java
public static void cachedThreadPool() {
    // 创建线程池
    ExecutorService threadPool = Executors.newCachedThreadPool();
    // 执行任务
    for (int i = 0; i < 10; i++) {
        threadPool.execute(() -> {
            System.out.println("任务被执行,线程:" + Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
            }
        });
    }
}
```

### 4. SingleThreadExecutor

```java
public static void singleThreadExecutor() {
    // 创建线程池
    ExecutorService threadPool = Executors.newSingleThreadExecutor();
    // 执行任务
    for (int i = 0; i < 10; i++) {
        final int index = i;
        threadPool.execute(() -> {
            System.out.println(index + ":任务被执行");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
            }
        });
    }
}
```

### 5. ScheduledThreadPool

```java
public static void scheduledThreadPool() {
    // 创建线程池
    ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(5);
    // 添加定时执行任务(1s 后执行)
    System.out.println("添加任务,时间:" + new Date());
    threadPool.schedule(() -> {
        System.out.println("任务被执行,时间:" + new Date());
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
    }, 1, TimeUnit.SECONDS);
}
```

### 6. SingleThreadScheduledExecutor

```java
public static void SingleThreadScheduledExecutor() {
    // 创建线程池
    ScheduledExecutorService threadPool = Executors.newSingleThreadScheduledExecutor();
    // 添加定时执行任务(2s 后执行)
    System.out.println("添加任务,时间:" + new Date());
    threadPool.schedule(() -> {
        System.out.println("任务被执行,时间:" + new Date());
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
    }, 2, TimeUnit.SECONDS);
}
```

### 7. WorkStealingPool

```java
public static void workStealingPool() {
    // 创建线程池
    ExecutorService threadPool = Executors.newWorkStealingPool();
    // 执行任务
    for (int i = 0; i < 10; i++) {
        final int index = i;
        threadPool.execute(() -> {
            System.out.println(index + " 被执行,线程名:" + Thread.currentThread().getName());
        });
    }
    // 确保任务执行完成
    while (!threadPool.isTerminated()) {
    }
}
```

## 线程池提交任务

### execute()

execute方法用于提交不需要返回值的任务，如实现Runnable接口的任务。

### submit()

submint用于提交有返回值的任务，如实现Callable接口的任务，线程池会返回Future对象，通过Future可以判断任务是否执行成功。

## Executor框架

Executor主要用于线程管理，包括创建线程，分配任务

### 三大部分

- 任务：Runnable、Callable
    - 执行的任务需要实现Runnable接口或Callable接口。
- 执行：Executor
    - Executor和继承Executor的子类都可以用于执行任务。ExecutorService、ThreadPoolExecutor等。
- 异步计算结果：Future
    - 实现Runnable或Callable接口的任务使用submit方法交给ThreadPoolExecutor执行后会返回FutureTask对象

### ThreadPoolExecutor

ThreadPoolExecutor是Executor的核心继承类。

ThreadPoolExecutor的参数，也就是创建线程池的参数：

- corePoolSize：最小同时运行的线程数。
- maximumPoolSize：当队列中存放的任务达到队列最大容量时，当前可用时运行的最大线程数量。
- workQueue：如果没有空闲线程，新任务存放在队列中

## 饱和

饱和：当所有线程都在运行同时队列已满的情况

- 抛出异常，并拒绝新任务
- 不处理新任务，直接丢掉
- 丢去最早未处理的任务请求

# Java中锁的类型

> [图片来自](https://www.pdai.tech/md/java/thread/java-thread-x-lock-all.html)

![image-20221012155348418](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221012155348418.png)

## 乐观锁、悲观锁

- 悲观锁会在修改期间始终加锁，确保数据不被其他的线程修改。**属于锁定之后在操作资源**
    - 在Java中 `synchoronized`关键字 和 `ReentrantLock` 实现类都是悲观锁。
    - 适合写入数据的场景。
- 乐观锁在修改期间不会加锁，但会在修改之前判断数据是否被其他线程修改，如果没有被其他线程更新，就更新自己的数据，如果已经被其他线程更新了则退出。**属于直接操作资源**
    - 在Java中，乐观锁由CAS算法实现，在Java中例如原子类AtomicInteger自增属于乐观锁。
    - 适合读取数据的场景。

## 自旋锁、适应性自旋锁

### 自旋锁

当多个线程操作一个资源时，由于资源同时只能被一个线程操作，且资源操作时会被加锁。在资源被占有的情况下，其他线程无法访问该资源，就需要转换线程状态，由运行转向等待。

线程状态的改变，也就是上下文切换，会导致资源的浪费，有时有对资源的操作（加锁和释放锁）速度很快，没有必要让其他线程进入等待状态，此时可以让这个线程不放弃CPU时间片，重复尝试获取资源（默认10次），也就是让线程自旋，这样减少了线程状态切换带了的性能开销。

如果是非自旋锁，按照正常流程需要执行CPU切换状态，线程休眠、等待资源上的锁被释放在尝试对资源加锁。

### 适应性自旋锁

锁的自旋时间不固定，锁的时间取决于前一次在同一个锁上的自旋时间及锁拥有者的状态决定。

## 无锁、偏向锁、轻量级锁、重量级锁

指锁的状态

- 偏向锁：通过对比Mark Word解决加锁问题，避免CAS操作。
- 轻量级锁：通过CAS操作和自旋解决加锁问题。
- 重量级锁：除了拥有锁的线程之外的线程全部阻塞。

## 公平锁、非公平锁

- 公平锁是指多个线程按照申请锁的顺序获取锁。
- 非公平锁是福哦哥线程加锁是直接尝试获取锁，如果获取不到则添加到等待队列。

## 可重入锁、非可重入锁

可重入锁（递归锁），同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁。不会因为之前已经获取的锁没有释放而阻塞线程。

示例：

```java
public class Widget 
{
    public synchronized void doSomething() 
    {
        System.out.println("方法1执行...");
        doOthers();
    }

    public synchronized void doOthers() 
    {
        System.out.println("方法2执行...");
    }
}
```

- 可重用锁：doSomething调用doOthers时，如果是可重用锁，已经获得doSomething 的锁，在执行doOthers不需要释放doSomething的锁。

- 不可重用锁：如果是不可重用锁，如果要执行doOthers就要释放doSomething的锁。

## 独享锁（排他锁）、共享锁

- 独享锁：一次只能被一个线程持有。
- 共享锁：锁可以被多个线程持有。

# ThreadLocal

单个线程独有的本地变量存放

访问这个变量的每个线程都有这个变量的本地副本，可以使用get set方法获取默认值或将将至修改问当前副本的值。

# CAS

CAS（Compare-And-Swap）对比交换。

这是一条CPU原子指令，作用是让CPU先对比两个值是否相等，之后原子性地更新某个位置的值。

CAS操作是原子性的，所以多线程并发使用CAS更新数据时，可以不使用锁。

缺点：只能保证一个共享变量的原子性操作

## 使用CAS

基于Java中的原子类 `AtomicInteger`

```java
public class Test 
{
    private  AtomicInteger i = new AtomicInteger(0);
    
    public int add()
    {
        return i.addAndGet(1);
    }
}
```

## ABA问题

CAS在操作值时检查值是否发生变化，没有变化再更新，如果一个值原来是A，变成了B再次变成A，CAS进行检查时会发现该值没有发生变化，但是实际上变化了。

在变量前追加版本号解决该问题，每次更新后都将版本号+1，ABA -> 1A2B3A

# Atomic 原子类

原子是指操作不可中断，如果是多线程，原子操作不受其他线程影响。

## 原子类（12个）

### 基本类型

- AtomicInteger：整形原子类
- AtomicLong：长整形原子类
- AtomicBoolean：布尔原子类

### 数组类型

- AtomicIntegerArray：整形数组原子类
- AtomicLongArray：长整形数组原子类
- AtomicReferenceArray：引用类型数组原子类

### 引用类型

- AtomicReference：引用类型原子类
- AtomicStampedReference：原子更新带有版本号的引用类型
- AtomicMarkableReference：原子更新带有标记位的引用类型

### 对象属性修改类型

- AtomicIntengerFieldUpdater：原子更新整型字段的更新器
- AtomicLongFiledUpdater：原子更新长整型字段的更新器
- AtomicReferenceFieldUpdater：原子更新引用类型字段的更新器

# AQS

AQS（AbstractQueuedSynchronizer）抽象排队同步器。AQS是一个用来构建锁和同步器的框架。

## 核心思想

如果被请求的共享资源空闲，则将当前请求资源的线程设置为优先工作线程，并将共享资源设置为锁定状态。如果被请求的共享资源被占用，线程会被阻塞、唤醒。AQS会将阻塞线程加入队列中，等待资源释放。

## 资源共享方式

- 独享：只有一个线程能获取，如ReentrantLock
- 共享：多个线程可同时执行

