---
layout: post
title: "ThreadPoolExecutor 工作线程Worker自身锁设计"
subtitle: "详解Worker加锁设计方案与原理"
author: "iwts li"
date: 2022-01-15 22:33:00
categories: Java
header-style: text
tags:
  - Java
  - JUC
description: "Java ThreadPoolExecutor, Worker, 工作线程"
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

# 工作线程自身锁

Worker对象其实本身就是一把锁。这是个细节，Worker本身是实现了AQS的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301933953.png)

这里其实最主要的作用是工作线程的回收。虽然可以通过维护workers来完成对工作线程生命周期的管理，新建线程比较好理解，但是删除线程的时候，工作线程本身就是一种竞争资源了。回收的时候是可能恰好碰到调用的。

这里选择AQS的原因，其实可以看注释，这边简单翻译一部分：

> Worker类存在的主要意义就是为了维护线程的中断状态。因为正在执行任务的线程是不应该被中断的。在线程真正开始运行任务之前，为了抑制中断。所以把 Worker 的状态初始化为负数-1。

完全看不懂，这里从其他角度慢慢绕过来解释一下。

## 线程的中断与回收与加锁时机

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

# 不使用ReentrantLock和synchronized的原因

为啥要加锁，就是为了区分线程是否中断，而ReentrantLock和synchronized都有一个重要特性：可重入。因为可重入，那么这个锁就没有意义了，因为线程都是一个，既然可重入那么就是必然能获锁了。

所以选用AQS，手动删掉可重入的特性，实现互斥。

# tryAcquire实现不可重入

也可以看看重写AQS的获锁代码：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202406301944084.png)

虽然设置的excluiveOwnerThread，但是完全不用，就是直接CAS获锁，没有重入的特性。
