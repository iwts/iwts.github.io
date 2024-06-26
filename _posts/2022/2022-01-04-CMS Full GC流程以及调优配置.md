---
layout: post
title: "CMS Full GC流程以及调优配置"
author: "iwts li"
date: 2022-01-04 15:28:00
categories: JVM
header-style: text
tags:
  - JVM
  - Java
description: "JVM CMS GC, JVM 调优"
---

# CMS

CMS 收集器是以实现最短 STW 时间为目标的收集器，所以对于偏业务的后台开发而言，基本上都无脑选CMS了。

多线程收集器，工作在老年代，采用标记清除算法。比较特殊，其他两个老年代收集器都是标记整理。

# 标记清除算法流程

1. 初始标记（STW）。
   
   a. 仅标记 GC Roots 能关联的对象，速度很快。
2. 并发标记。
   
   a. GC Roots Tracing。
   
   b. 此时用户线程是在工作的。
3. 重新标记（STW）。
   
   a. 修正并发标记期间因用户线程继续运行而导致标记产生变动的那一部分对象的标记记录。
   
   b. STW时间稍长。
4. 并发清除。
   
   a. 此时用户线程在工作。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405262116493.png)

总体上看，可以认为 CMS 收集器的内存回收过程是与用户线程一起并发执行的。

在老年代达到80%时进行Full GC，默认6次Minor GC晋升到老年代。

# CMS的缺点以及对应配置调优

## 吞吐量低下问题

CMS 收集器对 CPU 资源非常敏感，因为需要分出回收线程，导致吞吐量下降。

CMS 默认启动的回收线程数是：
$$
回收线程数=\frac{CPU数量+3}{4}
$$

所以如果 CPU 数量只有一两个，那吞吐量就直接下降 50%+。

### 解决方案

没有什么解决方案，一般现在都是4/8 cores+的pod吧，理论上不太可能出现这个问题。顶不住就加机器配置，简单粗暴好用。

## 浮动垃圾导致的连续Full GC

CMS并发清理阶段多个用户线程在并行运行，所以在清理的同时工作线程也会持续产生新的垃圾。

这个垃圾就是所谓的浮动垃圾（Floating Garbage），只能在下一次 GC 时再清理掉。

取决于 CMS 的算法，这个问题是无法处理，所以可能出现 Concurrent Mode Failure 从而而导致另一次 Full GC 的产生。

### 解决方案

受限于 CMS 的算法，这个问题是无法解决的，只能避免。

并发处理时，用户线程也要继续运行，需要预留足够多的空间要确保用户线程正常执行，所以 CMS 不能等老年代满了再执行。

所以一个重要的预防点：老年代空间占用多少时触发CMS？这个比例很重要。

比例可以通过```-XX:CMSInitiatingOccupancyFraction```来设置。

至于比例到底怎么设，这跟具体业务有关，没有完美答案。一般来说只能是碰到问题，然后进行调整，每个业务都有适合自己的比例

#### 比例太高时的现象

设置太高，可能会导致在 CMS 运行期间预留的内存无法满足程序要求，并发执行阶段失败，出现Concurrent Mode Failure。

这时会启用 Serial Old 收集器来重新进行老年代的收集，作为备份。

Serial Old 收集器是单线程收集器，会导致 STW 变的非常长。

#### 比例太低时的现象

设置太低，频繁GC

## 内存碎片整理导致Full GC时间过长

CMS 采用的是标记清除法，会产生大量的内存碎片，整体内存空间利用率太低，大内存对象分配时可能会经常触发 Full GC。

所以CMS会在Full GC时做内存碎片的合并整理，通过设置```-XX:+UseCMSCompactAtFullCollection```开启（默认是开启的）。

那么就带来了另一个问题：内存整理会花费时间，从而导致STW时间会变长。导致Full GC时间过长。

### 解决方案

设置参数```-XX:CMSFullGCsBeforeCompation```，设置执行多少次不压缩的 Full GC 后执行一次带压缩的Full GC。

从而保证只会偶尔出现一次长时间Full GC。

至于参数怎么设置，依旧与你的服务有关，没有固定答案。
