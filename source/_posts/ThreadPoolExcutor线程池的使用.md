---
title: ThreadPoolExcutor线程池的使用
date: 2017-05-23 23:39:30
tags: ['多线程','并发']
categories: JAVA基础
---

### 1.基本概念
线程池类为 Java.util.concurrent.ThreadPoolExecutor，常用构造方法为：
```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,
long keepAliveTime, TimeUnit unit,
BlockingQueue<Runnable> workQueue,
RejectedExecutionHandler handler)
corePoolSize： 线程池维护线程的最少数量
maximumPoolSize：线程池维护线程的最大数量
keepAliveTime： 线程池维护线程所允许的空闲时间
unit： 线程池维护线程所允许的空闲时间的单位
workQueue： 线程池所使用的缓冲队列
handler： 线程池对拒绝任务的处理策略
```
一个任务通过 execute(Runnable)方法被添加到线程池，任务就是一个 Runnable类型的对象，任务的执行方法就是 Runnable类型对象的run()方法。
当一个任务通过execute(Runnable)方法欲添加到线程池时：
- 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
- 如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 blockQueue未满，那么任务被放入缓冲队列。
- 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
- 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。

也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

work queue有以下几种实现：
1. ArrayBlockingQueue :  有界的数组队列
2. LinkedBlockingQueue : 可支持有界/无界的队列，使用链表实现
3. PriorityBlockingQueue : 优先队列，可以针对任务排序
4. SynchronousQueue : 队列长度为1的队列，和Array有点区别就是：client thread提交到block queue会是一个阻塞过程，直到有一个worker thread连接上来poll task。

workQueue常用的是：java.util.concurrent.ArrayBlockingQueue

handler有四个选择：
```java
ThreadPoolExecutor.AbortPolicy()
抛出java.util.concurrent.RejectedExecutionException异常
ThreadPoolExecutor.CallerRunsPolicy()
直接让原先的client thread做为worker线程，进行执行
ThreadPoolExecutor.DiscardOldestPolicy()
丢弃最早入队列的的任务
ThreadPoolExecutor.DiscardPolicy()
抛弃当前的任务
```
### 2.建议
#### 2.1.ThreadPoolExecutor允许你提供一个BlockingQueue来持有等待执行的任务。任务排队有3种基本方法：无限队列、有限队列和同步移交。
#### 2.2.newFixedThreadPool和newSingleThreadExectuor默认使用的是一个无限的 LinkedBlockingQueue。如果所有的工作者线程都处于忙碌状态，任务会在队列中等候。如果任务持续快速到达，超过了它们被执行的速度，队列也会无限制地增加。稳妥的策略是使用有限队列，比如ArrayBlockingQueue或有限的LinkedBlockingQueue以及 PriorityBlockingQueue。
#### 2.3.对于庞大或无限的池，可以使用SynchronousQueue，完全绕开队列，直接将任务由生产者交给工作者线程
#### 2.4.可以使用PriorityBlockingQueue通过优先级安排任务
#### 2.5.善用blockqueue和reject组合. 
这里要重点推荐下CallsRun的Rejected Handler，从字面意思就是让调用者自己来运行。
我们经常会在线上使用一些线程池做异步处理 将原本串行的请求都变为了并行操作，但过多的并行会增加系统的负载(比如软中断，上下文切换)。所以肯定需要对线程池做一个size限制。但是为了引入异步操作后，避免因在block queue的等待时间过长，所以需要在队列满的时，执行一个callsRun的策略，并行的操作又转为一个串行处理，这样就可以保证尽量少的延迟影响。

所以建议：   maximumPoolSize >= corePoolSize =期望的最大线程数  RejectExecutionHandler = CallsRun ,  blockqueue size = 2 * poolSize (为啥是2倍poolSize，主要一个考虑就是瞬间高峰处理，允许一个thread等待一个runnable任务)

#### 2.6.队列维护
方法 getQueue() 允许出于监控和调试目的而访问工作队列。强烈反对出于其他任何目的而使用此方法。remove(java.lang.Runnable) 和 purge() 这两种方法可用于在取消大量已排队任务时帮助进行存储回收。

### 3.扩展
ThreadPoolExecutor是可扩展的，通过查看源码可以发现，它提供了几个可以在子类化中改写的方法：beforeExecute,afterExecute,terminated.
```java
protected void beforeExecute(Thread t, Runnable r) { }  
protected void afterExecute(Runnable r, Throwable t) { }  
protected void terminated() { }  
```
可以注意到，这三个方法都是protected的空方法，摆明了是让子类扩展的嘛。
在执行任务的线程中将调用beforeExecute和afterExecute等方法，在这些方法中还可以添加日志、计时、监视或者统计信息收集的功能。无论任务是从run中正常返回，还是抛出一个异常而返回，afterExecute都会被调用。如果任务在完成后带有一个Error，那么就不会调用afterExecute。如果beforeExecute抛出一个RuntimeException，那么任务将不被执行，并且afterExecute也不会被调用。
在线程池完成关闭时调用terminated，也就是在所有任务都已经完成并且所有工作者线程也已经关闭后，terminated可以用来释放Executor在其生命周期里分配的各种资源，此外还可以执行发送通知、记录日志或者手机finalize统计等操作。

下面就以给线程池添加统计信息为例（添加日志和计时等功能）：
```java
package com.threadPool;  
  
import java.util.concurrent.BlockingQueue;  
import java.util.concurrent.ThreadPoolExecutor;  
import java.util.concurrent.TimeUnit;  
import java.util.concurrent.atomic.AtomicLong;  
import java.util.logging.Logger;  
  
public class TimingThreadPool extends ThreadPoolExecutor  
{  
    private final ThreadLocal<Long> startTime = new ThreadLocal<Long>();  
    private final Logger log = Logger.getAnonymousLogger();  
    private final AtomicLong numTasks = new AtomicLong();  
    private final AtomicLong totalTime = new AtomicLong();  
      
    public TimingThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,  
            BlockingQueue<Runnable> workQueue)  
    {  
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);  
    }  
  
    protected void beforeExecute(Thread t, Runnable r){  
        super.beforeExecute(t, r);  
        log.info(String.format("Thread %s: start %s", t,r));  
        startTime.set(System.nanoTime());  
    }  
      
    protected void afterExecute(Runnable r, Throwable t){  
        try{  
            long endTime = System.nanoTime();  
            long taskTime = endTime-startTime.get();  
            numTasks.incrementAndGet();  
            totalTime.addAndGet(taskTime);  
            log.info(String.format("Thread %s: end %s, time=%dns", t,r,taskTime));  
        }  
        finally  
        {  
            super.afterExecute(r, t);  
        }  
    }  
      
    protected void terminated()  
    {  
        try  
        {  
            log.info(String.format("Terminated: avg time=%dns",totalTime.get()/numTasks.get()));  
        }  
        finally  
        {  
            super.terminated();  
        }  
    }  
}  


```
下面写一个测试类，参考运行效果：
```java
package com.threadPool;  

import java.util.concurrent.SynchronousQueue;  
import java.util.concurrent.ThreadPoolExecutor;  
import java.util.concurrent.TimeUnit;  
  
public class CheckTimingThreadPool  
{  
    public static void main(String[] args)  
    {  
        ThreadPoolExecutor  exec = new TimingThreadPool(0, Integer.MAX_VALUE,  
                60L, TimeUnit.SECONDS,  
                new SynchronousQueue<Runnable>());  
        exec.execute(new DoSomething(5));  
        exec.execute(new DoSomething(4));  
        exec.execute(new DoSomething(3));  
        exec.execute(new DoSomething(2));  
        exec.execute(new DoSomething(1));  
        exec.shutdown();  
    }  
  
}  
  
class DoSomething implements Runnable{  
    private int sleepTime;  
    public DoSomething(int sleepTime)  
    {  
        this.sleepTime = sleepTime;  
    }  
    @Override  
    public void run()  
    {  
        System.out.println(Thread.currentThread().getName()+" is running.");  
        try  
        {  
            TimeUnit.SECONDS.sleep(sleepTime);  
        }  
        catch (InterruptedException e)  
        {  
            e.printStackTrace();  
        }  
    }  
      
} 
```