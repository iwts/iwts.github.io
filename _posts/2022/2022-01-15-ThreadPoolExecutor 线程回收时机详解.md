---
layout: post
title: "ThreadPoolExecutor 线程回收时机详解"
author: "iwts li"
date: 2022-01-15 22:35:00
categories: Java
header-style: text
tags:
  - Java
  - JUC
description: "Java ThreadPoolExecutor, Worker"
---

# 总集

想要完整了解下ThreadPoolExecutor？可以参考：

[基于源码详解ThreadPoolExecutor实现原理 | iwts's blog](https://iwts.github.io/blog/%E5%9F%BA%E4%BA%8E%E6%BA%90%E7%A0%81%E8%AF%A6%E8%A7%A3ThreadPoolExecutor%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)

# Worker-工作线程管理

线程池设计了内部类```Worker```，主要是用来管理新建的线程，除了监控，核心的方法是：

1. 执行。
2. 申请任务。

此外还包括回收等线程监控类型方法。

由于一个工作线程对象，其中有一个具体的线程，那么本质上是不需要加锁的。竞争资源是任务队列，而任务队列由阻塞队列来实现。

可以看Worker的设计：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301920023.png)

# 线程生命周期管理

线程池需要管理线程的生命周期，需要在线程长时间不运行的时候进行回收。

线程池使用一张Hash表去持有线程的引用，这样可以通过添加引用、移除引用这样的操作来控制线程的生命周期。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301931482.png)

可以看到，workers是HashSet，那么问题来了，线程池有大量的工作线程，频繁创建/清除线程的时候，用线程不安全的HashSet必然是有并发安全问题的。

所以线程池要求在操作workers的时候，都需要获锁，根据该锁对workers进行操作：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301932769.png)

也就是说，在工作线程的创建/销毁，都要加上这个锁，例如工作线程的创建：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301932219.png)

## 工作线程的回收

这里比较复杂，慢慢聊。

### 工作线程自身锁

Worker对象其实本身就是一把锁。这是个细节，Worker本身是实现了AQS的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301933953.png)

这里其实最主要的作用是工作线程的回收。虽然可以通过维护workers来完成对工作线程生命周期的管理，新建线程比较好理解，但是删除线程的时候，工作线程本身就是一种竞争资源了。回收的时候是可能恰好碰到调用的。

这里选择AQS的原因，其实可以看注释，这边简单翻译一部分：

> Worker类存在的主要意义就是为了维护线程的中断状态。因为正在执行任务的线程是不应该被中断的。在线程真正开始运行任务之前，为了抑制中断。所以把 Worker 的状态初始化为负数-1。

完全看不懂，这里从其他角度慢慢绕过来解释一下。

### 线程的中断与回收

解释这个问题，首先看下Worker自身是从哪里调用锁的：

1. 工作线程处理前后加锁。
2. 工作线程尝试中断时尝试获锁。

第一个看代码```runWorker()```：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301934524.png)

也就是说，当前线程如果在处理，那么本身是给自己加锁的。
第二个看代码```interruptIdleWorkers()```：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301934290.png)

这里是不是就有点恍然大悟的意思了。

工作线程本身实现AQS，将自身当作竞争对象。

那么工作线程工作的时候，加锁，锁住自己，那么```interruptIdleWorkers()```方法在执行的时候，如果能获取锁，就说明一个问题：此时当前线程是没有在工作的。那么就会被中断掉。

为了实现这个功能，就只能选择不可重入锁，所以自己实现了AQS来实现这个特性。具体可以看代码实现。

基本可以推测到，```interruptIdleWorkers()```这个方法就是回收方法，那么其调用时机是什么？

### 线程回收时机

#### 一个重要的回收时机-keepAliveTime

这里单独拉出来聊了，比较经典。

八股文一般说：keepAliveTime是线程存活时间，如果当前线程池线程数量大于核心池的时候，如果一个线程超过keepAliveTime没有获取到任务，则会触发线程回收。

这里聊聊相关源码。首先看基础的任务申请：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301936463.png)

这里如果设置了超时时间的情况下，请求任务队列是调用的```poll()```方法，并指定了keepAliveTime。那么这个方法的意思就是，阻塞这么长时间，超过时间后直接返回null。

所以这里就对应到八股文了，如果此时```poll()```返回空，那么就是说当前队列里什么数据都没有，那么这里其实就是说明：该线程等待了keepAliveTime都没有获取到数据，也就是说这段时间全部是空闲。可以回收了。

而这里只是设置了timedOut标记，留给上层来处理：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301937146.png)

这里判定之后返回个null。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301937445.png)

直接跳出线程执行```run()```方法，在finally块中触发线程回收。```processWorderExit()```方法的底层就是下面的```tryTerminate()```了，会直接进行回收。

#### tryTerminate()

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301939225.png)

核心回收方法，根据其调用可以梳理出正常运行中的回收时机：

1. 工作线程创建失败时：```addWorkerFailed()```。
2. ```runWorker()```方法退出时。正常来说```runWorker()```方法是一个自旋，只有在任务申请失败时才会退出自旋。那么这个时机就是指任务队列已经清空了：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301939797.png)
总体流程为：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301940790.png)

3. shutdown()。可以看到执行了两次。
4. shutdownNow()。
5. remove()，移除任务时顺便执行一次。
6. purge()，todo。

#### interruptIdleWorkers

一般是针对线程池本身参数进行操作的时候，会触发回收，看其调用方式，可以梳理出来全部的线程回收时机：

1. shutdown()。
2. 设置核心池大小的时候，如果当前线程池线程数量大于核心池数量大小，执行一次回收：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301941282.png)
3. 设置允许核心池超时时，执行一次回收：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301941422.png)
4. 设置最大池数量时，如果当前线程池线程数量大于最大池数量，执行一次回收：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301942119.png)
5. 设置线程池线程存活时间时，如果设置变小了，那么执行一次回收：
![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301942104.png)
