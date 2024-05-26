---
layout: post
title: "JVM CMS 在Full GC时针对跨代引用的优化"
author: "iwts li"
date: 2022-01-30 21:09:00
categories: JVM
header-style: text
tags:
  - JVM
  - Java
description: "JVM CMS, Full GC, 跨代引用"
---

# 跨代引用问题

Full GC慢的一个很重要的问题：跨代引用。

简单描述就是：

现代JVM一般是根据对象存活的特性进行了分代，提高了垃圾收集的效率。但是像在回收新生代的时候，有可能有老年代的对象引用了新生代对象，所以老年代也需要作为根，但是如果扫描整个老年代的话效率就又降低了。

这个就是跨代引用的问题。

## 跨代引用对Full GC的影响

由于跨代引用的扫描问题，导致Full GC时，如果此时新生代的数据量很大，会导致扫描时间显著增加。

# CMS 对跨代引用的优化

最简单粗暴的方法：Full GC前强制走一个Minor GC，那么新生代数据降到很低的值，就能解决这个问题。而CMS也是基本采用这个方案。

## CMS 并发预清理

CMS在Remark前增加了一个可中断的并发预清理（CMS-concurrent-abortable-preclean），该阶段主要工作仍然是并发标记对象是否存活，只是这个过程可被中断。

此阶段在Eden区使用超过2M（默认阈值，可修改）时启动，如果此阶段执行时等到了Minor GC，那么跨代引用的对象将会跟随Minor GC被清除掉，Reamark阶段需要扫描的对象就少了。

此外，CMS为了避免这个阶段没有等到Minor GC而陷入无限等待，提供了参数```CMSMaxAbortablePrecleanTime``` ，默认为5s，如果可中断的预清理执行超过5s，不管发没发生Minor GC，都会终止此阶段，进入Remark。

所以，CMS在Remark前也不一定会执行Minor GC，还是有风险。CMS提供```CMSScavengeBeforeRemark```参数，设置后可以保证Remark前强制进行一次Minor GC。
