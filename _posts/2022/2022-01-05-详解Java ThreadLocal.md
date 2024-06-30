---
layout: post
title: "详解Java ThreadLocal"
author: "iwts li"
date: 2022-01-05 04:48:00
categories: Java
header-style: text
tags:
  - Java
  - Java 多线程
  - JUC
description: "Java ThreadLocal"
---

# Java ThreadLocal

ThreadLocal提供了线程内存储变量的能力，这些变量不同之处在于每一个线程读取的变量是对应的互相独立的。通过get和set方法就可以得到当前线程对应的值。

# TreadLocal存储模型

ThreadLocal的静态内部类ThreadLocalMap为每个Thread都维护了一个数组table，ThreadLocal确定了一个数组下标，而这个下标就是value存储的对应位置。
这样看太抽象了，具体数据结构可以从存储方法开始看。

## 根据set方法探究ThreadLocal存储模型

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270225195.png)

看上去简单粗暴，先是获取线程，然后看能不能获取数据结构，能获取就直接set，获取不到就初始化。

那么这里指向了一个核心方法：ThreadLocalMap是怎么获得的。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270226517.png)

这个东西竟然是存储在线程对象里面的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270226196.png)

等于说TreadLocal的核心内部类，是存在一个线程中的。
这个时候再回过头来看ThreadLocalMap：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270226851.png)

这个Entry继承了WeakReference，代表他是一个弱引用，而其泛型则是ThreadLocal，那么说明了单个Entry节点，底层是一个kv结构，key是ThreadLocal的弱引用，value是一个Object。

但是这个只是Entry节点，ThreadLocalMap的底层并不是KV结构，而是一个Entry数组。

这里再继续看```set()```方法是怎么处理的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270227136.png)

可以看出来，Entry的key就是这个具体的ThreadLocal对象，value是具体设置的value。

那么这个下标怎么计算的？

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270227479.png)

这里与计算比较类似HashMap，保证数组长度是2的幂，这样与运算直接就是取模了，具体内容参考下面。

综上，可以得到ThreadLocal究竟是怎么存的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270228737.png)

注意，key存储的是ThreadLocal的弱引用，那么扩展到JVM内存模型下：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270228446.png)

ThreadLocal本身的引用是正常放在栈中的，但是由于WeakReference，所以key对ThreadLocal的引用是虚引用。这也是OOM的主要原因。

## Thread中threadLocals

整个ThreadLocal的数据都是维护在单个线程的属性中，所以保证了线程间的隔离，因为这玩意就是线程自己的一个变量。

## ThreadLocalMap

ThreadLocal的内部类，其中定义了Entry节点，存储具体的数据。而本身提供了Entry数组，用于存储线程下全部的ThreadLocal数据。

此外提供了一系列数组操作方法，以及扩容等操作。

### Entry

内部类Entry，继承了WeakReference，其内部设置属性value，实现了一个类似KV结构的数据结构，保证了key是ThreadLocal的一个弱引用，value是具体的数值。

# ThreadLocalMap的维护

## getMap()

等于从线程中获取线程本身的ThreadLocalMap对象。

## ThreadLocalMap的初始化

```java.lang.ThreadLocal#createMap```

直接走了构造方法：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270230599.png)

初始化了Entry数组。初始化数组长度为16，并且设置了扩容阈值threshold。具体看下面。

## 扩容机制

类似HashMap扩容，初始化长度为16，后续扩容的时候直接*2，保证长度是2的幂。只有在超过阈值的时候才会扩容。

阈值threshold的维护，为当前数组长度的2/3。

跟HashMap一样，扩容后需要对旧值重新hash，定位到确定的下标。相比HashMap简单多了。

## ThreadLocal对象hash定位下标

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270230777.png)

算法看起来还是比较简单的。了解HashMap这里没啥难度，由于长度必然是2的幂，这样与计算也是和取模的效果是一样的。

主要是这个threadLocalHashCode是什么。

这里算是一个小优化了，主要获取方法：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270231222.png)

这里走CAS执行一次自增，增量为常量0x61c88647。

这样，0x61c88647是斐波那契散列乘数，那么后续的自增导致hash出来的结果分布会比较均匀，可以很大程度上避免hash冲突，下面是15次操作获取的hash值

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270231816.png)

### hash冲突

由于ThreadLocalMap的底层是一个数组，设计上也不想太复杂，所以没有采用HashMap的拉链法，而是采用线性探测来解决hash冲突。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405270232665.png)

非常简单，其实就是+1，看是否越界，越界设置0。其实这里改成取模更加简洁。

# ThreadLocal弱引用与内存泄漏

回到最上面Entry的设计，继承了WeakReference，表明了key的对象是个弱引用。

那么在ThreadLocal执行完毕后，如果没有及时清除，就会导致ThreadLocal对象没有具体的引用对象。那么就会被GC回收掉。

参考上面的图，GC回收的是堆，堆中的这个key，他引用的是ThreadLocal，也就是说这个key的引用，是会被GC回收掉的。

而Entry中value是个普通的Object，那么value本身是不会被回收的。
这样在大量操作后，就会导致value无法回收，导致内存泄漏。

## 解决方案

ThreadLocal使用结束后及时remove()。

而JDK为了解决这个问题，本身也或多或少在帮助我们回收。

remove()的时候会主动触发，而get()、set()在执行的时候也会间接执行：expungeStaleEntry()方法。

这个方法会主动清除所有Entry中key为null的对象。

### 强引用解决方案

比较奇葩，感觉用的不多。

即利用static修饰ThreadLocal，这样就能保证ThreadLocal对象是强引用，从根本上解决问题。

## 为什么选择用弱引用

依然是OOM问题。

由于ThreadLocalMap的生命周期跟Thread一样长，如果是强引用，那么手动删除对应key的情况下，仍然会导致内存泄漏。

但是使用弱引用可以多一层保障：弱引用ThreadLocal被清理后key为null，对应的value在下一次ThreadLocalMap调用set、get、remove的时候可能会被清除。

弱引用虽然会导致OOM，但是强引用不仅会导致OOM，还会更多。

# ThreadLocal API

都比较简单感觉没什么好聊的。

需要注意的是hash冲突问题，由于hash冲突的解决方案是线性探测，所以在get的时候，需要对比key是否一致，如果不一致可能是hash冲突，需要利用线性探测，循环遍历一轮数组。
