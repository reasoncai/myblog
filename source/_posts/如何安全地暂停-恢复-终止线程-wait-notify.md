---
title: 如何安全地暂停-恢复-终止线程(wait/notify)
date: 2017-06-17 20:47:13
tags: ['多线程','并发']
categories:
---

#### 1.过期的suspend(),resume(),stop()
这三个方法完成了线程的暂停、恢复和终止工作。但是是过期的方法，不建议使用。原因：以suspend()方法为例，在调用后，线程不会释放已经占用的资源（比如锁），而是占有着资源进入睡眠状态，这样容易引起死锁问题。同样，stop()方法在终止一个线程的时候不会保证线程的资源正常释放，通常是没有给予线程完成资源释放的工作的机会，因此会导致程序可能工作在不确定状态下。

#### 2.用标志位或中断来替代stop()
```java
package com.cai.javademo.concurency;

import java.util.concurrent.TimeUnit;

/**
 * 安全地终止线程
 * Created by reason on 17/6/17.
 */
public class Shutdown {
    public static void main(String[] args) throws InterruptedException {
        Runner runner1 = new Runner();
        Thread countThread = new Thread(runner1);
        countThread.start();
        //方法1：中断
        //睡眠1秒，main线程对countThread进行中断，使其能感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();

        Runner runner2 = new Runner();
        countThread = new Thread(runner2);
        countThread.start();
        //方法2：设置标志位（推荐）
        //睡眠1秒，main线程对countThread进行取消，使其感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        runner2.cancel();
    }

    private static class Runner implements Runnable {
        private long i;
        private volatile boolean on = true;
        @Override
        public void run() {
            while(on && !Thread.currentThread().isInterrupted()){
                i ++;
            }
            System.out.println("count i = " + i);
        }

        public void cancel() {
            on = false;
        }
    }
}

```
#### 3.用等待/通知机制(wait/notify)来替代suspend()和resume()
##### 等待/通知的编程范式
- 等待方遵循如下原则：

1）获取对象的锁

2）如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。

3）条件满足则执行对应的逻辑
```
对应的伪代码如下：
synchronized(对象){
    while(条件不满足){
        对象.wait();
    }
    对应的处理逻辑
}
```
- 通知方遵循如下原则：

1）获取对象的锁

2）改变条件

3）通知所有等待在对象上的线程
```
对应的伪代码如下：
synchronized(对象){
    改变条件
    对象.notifyAll();
}
```
##### 3.1不带超时时间
```java
package com.cai.javademo.concurency;

import java.util.Date;

/**
 * Created by reason on 17/6/17.
 */
public class WaitNotify {
    private static boolean flag = true;
    private static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        Thread.sleep(1000);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();

    }

    private static class Wait implements Runnable {

        @Override
        public void run() {
            //加锁,拥有lock的Monitor
            synchronized (lock) {
                //当条件不满足时，继续wait,同时释放了lock的锁
                while (flag){
                    System.out.println(Thread.currentThread() + " flag is true. wait @ "+ new Date());
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ "+ new Date());
            }
        }
    }

    private static class Notify implements Runnable{

        @Override
        public void run() {
            //加锁,拥有lock的Monitor
            synchronized (lock){
                //获取lock的锁，然后进行通知，通知时不会释放lock的锁
                //直到当前线程释放了lock后，waitThread才能从wait方法中返回
                System.out.println(Thread.currentThread()+" hold lock.notify @ "+new Date());
                flag = false;
                lock.notifyAll();
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //Thread.yield();
            //再次加锁
            synchronized (lock){
                System.out.println(Thread.currentThread()+" hold lock again. @"+new Date());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```
运行结果：
```
Thread[WaitThread,5,main] flag is true. wait @ Sat Jun 17 20:33:34 CST 2017
Thread[NotifyThread,5,main] hold lock.notify @ Sat Jun 17 20:33:35 CST 2017
Thread[NotifyThread,5,main] hold lock again. @Sat Jun 17 20:33:37 CST 2017
Thread[WaitThread,5,main] flag is false. running @ Sat Jun 17 20:33:38 CST 2017

```
##### 3.2等待超时模式
```java
package com.cai.javademo.concurency;

import java.util.Date;

/**
 * Created by reason on 17/6/17.
 */
public class WaitNotifyTimeOut {
    private static boolean flag = true;
    private static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        //睡眠时间大于超时时间，使其超时
        Thread.sleep(5000);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();

    }

    private static class Wait implements Runnable {
        //超时时间
        private long remaining = 2000 ;

        @Override
        public void run() {
            //加锁,拥有lock的Monitor
            synchronized (lock) {
                    long future = System.currentTimeMillis() + remaining;
                    while(true && remaining > 0) {
                        System.out.println(Thread.currentThread() + " wait @ " + new Date() + flag);
                        try {
                            //设置2秒超时返回
                            lock.wait(remaining);
                            remaining = future - System.currentTimeMillis();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                System.out.println(Thread.currentThread() + " running @ "+ new Date()+flag);
            }
        }
    }

    private static class Notify implements Runnable{

        @Override
        public void run() {
            //加锁,拥有lock的Monitor
            synchronized (lock){
                //获取lock的锁，然后进行通知，通知时不会释放lock的锁
                //直到当前线程释放了lock后，waitThread才能从wait方法中返回
                System.out.println(Thread.currentThread()+" hold lock.notify @ "+new Date());
                flag = false;
                lock.notifyAll();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}

```
运行结果：
```
Thread[WaitThread,5,main] wait @ Sat Jun 17 20:36:07 CST 2017true
Thread[WaitThread,5,main] running @ Sat Jun 17 20:36:09 CST 2017true
Thread[NotifyThread,5,main] hold lock.notify @ Sat Jun 17 20:36:12 CST 2017
```