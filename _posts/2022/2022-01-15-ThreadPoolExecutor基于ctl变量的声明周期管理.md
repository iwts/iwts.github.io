---
layout: post
title: "ThreadPoolExecutor基于ctl变量的声明周期管理"
subtitle: "详解ctl变量运行原理"
author: "iwts li"
date: 2022-01-15 22:34:00
categories: Java
header-style: text
tags:
  - Java
  - JUC
description: "Java ThreadPoolExecutor, 线程池生命周期, ctl变量"
---

# 总集

想要完整了解下ThreadPoolExecutor？可以参考：

[基于源码详解ThreadPoolExecutor实现原理 | iwts's blog](https://iwts.github.io/blog/%E5%9F%BA%E4%BA%8E%E6%BA%90%E7%A0%81%E8%AF%A6%E8%A7%A3ThreadPoolExecutor%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)

# ctl字段的应用

线程池内部使用一个变量```ctl```维护两个值：
1. 运行状态（runState）
2. 线程数量 (workerCount)

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405290013753.png)

在具体实现中，就是进行位运算：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405290013235.png)

```COUNT_BITS```如果是32位的话，那么结合下面的一套左移、与、非的位运算，可以总结为：

1. ctl的高3位保存runState，即运行状态。
2. ctl的低29位保存workerCount，即有效线程数量。

除了ctl解析方法，还提供ctl计算方法，即根据runState和workerCount，计算出ctl值。

这样合并的好处是，操作的时候单锁就可以处理了（CAS也非常方便），位运算速度也快。
