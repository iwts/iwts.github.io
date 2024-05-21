---
layout: post
title: "Java StringBuilder详解以及与String、StringBuffer区别"
author: "iwts li"
date: 2018-10-22 20:47:30
categories: Java
header-style: text
tags:
  - Java
description: "Java StringBuilder详解以及与String、StringBuffer区别"
---

# StringBuilder

就像我们在Java入门教材中写的，在介绍String的时候写的是“字符串常量”，String实际上就是一个不可变的对象。每次使用String实际上是创建了一个不可变的对象，而改变这个String的值实际上是对这个字符串引用更换了新的地址，用图来表示：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405212118258.png)

原来的字符串仍然存在在内存之中。可以使用代码来测试一下：

```java
public class Main {
    public static void main(String[] args) throws Exception{
        String str = "name";
        System.out.println(str.hashCode());
        str = "new name";
        System.out.println(str.hashCode());
    }
}
```

那么对于StringBuilder而言，从命名我们也可以看出来：String的“建设者”，从Java SE 8的API中，官方也给出了这样的说法：

> A mutable sequence of characters. 

StringBuilder是一个字符串变量，相比与String，我们在对StringBuilder操作的时候，是对一个变量进行操作，这样对于JVM而言就会减轻内存占用。

StringBuilder引用指向的内存地址是不会变化的。

所以，相比与String，利用StringBuilder来进行字符串操作速度要比String快很多，所以例如字符串的反转、插入、拼接等操作，都推荐使用StringBuilder，最后转成String类型就可以了。

# StringBuilder的构造方法

最常见的是这两个：

```java
StringBuilder sb = new StringBuilder();

String str = "string";
StringBuilder sb = new StringBuilder(str);
```

默认的无参构造方法会直接创建一个StringBuilder对象，因为是可变序列，所以自然有容量这个说法，默认情况下是16个字符。也支持将一个String类型的字符串转化为StringBuilder。

既然有默认容量，自然也可以自己声明容量：

```java
StringBuilder(int capacity);
```

还有一种构造方法比较少用，是将CharSequence类型转化为StringBuilder，方法也是传进CharSequence对象即可。

# StringBuilder常用方法

## append

跟String一样，append适用于字符串、字符间的拼接。有多个重载方法适用于很多情况。常用的、看下就知道是干嘛的方法：

```java
append(char c)
append(char[] str)
append(CharSequence s)
append(CharSequence s, int start, int end)
append(double d)
append(float f)
append(int i)
append(long lng)
append(String str)
append(StringBuffer sb)
```

就是正常的拼接了，参数指定了如何拼接。```append()```方法返回值全部是一个StringBuilder类型，返回的是处理以后的StringBuilder对象，但实际上原串也被处理了。

比较特殊的方法有这些，注释是其作用：

```java
// 拼接一个boolean类型，其实就是拼接true或者false
append(boolean b)
// 对一个char数组，指定拼接的开始下标与长度，从offset开始，总共len个字符
append(char[] str, int offset, int len)
// 将Object的引用地址拼接上
append(Object obj)
```

例子：

```java
sb.append(true);
System.out.println(sb);
sb.append(chs,1,3);
System.out.println(sb);
sb.append(t);
System.out.println(sb);
```

输出：

```text
nametrue
nametruebcd
nametruebcdme.iwts.Test@1d44bcfa
```

可以看到，参数是boolean的时候，输入一个boolean类型，转化为字符串拼接上。而char数组则是从offset下标开始，拼接len个字符（包括offset）。

参数是对象的，则是拼接了该对象的引用地址。这里我特地全部在sb的基础上操作，可以看到，sb本身就进行了对应的操作，当然返回值返回的是操作后的StringBuilder对象。

## insert
        
也是有很多的重载方法：

```java
insert(int offset, boolean b)
insert(int offset, char c)
insert(int offset, char[] str)
insert(int index, char[] str, int offset, int len)
insert(int dstOffset, CharSequence s)
insert(int dstOffset, CharSequence s, int start, int end)
insert(int offset, double d)
insert(int offset, float f)
insert(int offset, int i)
insert(int offset, long l)
insert(int offset, Object obj)
insert(int offset, String str)
```

这里有一点，可以看到几乎所有的方法第一个参数都是一个int类型的数，这表示从StringBuilder的第offset个下标开始插入，与CharSequence对象相关的，dstOffset也是一样的意思，例如：

```java
StringBuilder sb = new StringBuilder("name");
int a = 123;
sb.insert(1,a);
System.out.println(sb);
```

输出：

```text
n123ame
```

如果上面append的看完的话，下面insert的基本都能了解是什么作用了，当然这里有一点，确实是不能指定String类型只插入一部分，例如"abcde"，就只能全插进去，不能设定插入其子串。但是对于char数组可以选择插入的范围：

```java
StringBuilder sb = new StringBuilder("name");
char[] chs = {'a','b','c','d','e'};
sb.insert(1,chs,2,3);
System.out.println(sb);
```

输出：

```text
ncdeame
```

## 其他

这些都是非常常用的方法：

```java
char charAt(int index)
int length()
StringBuilder reverse()
String substring(int start)
String substring(int start, int end)
StringBuilder delete(int start, int end)
StringBuilder deleteCharAt(int index)
StringBuilder replace(int start, int end, String str)
String toString()
```

这些是看一眼就知道是啥的方法，在String中也有，都在使用。substring是获取子串，这里返回的String类型而不是StringBuilder类型。这里注意一点，end这个参数跟正常思维是不太一样的，例如：

```java
StringBuilder sb = new StringBuilder("name");
sb.delete(1,2);
System.out.println(sb);
```

输出：

```text
nme
```

可以看到，删除的范围是从下标start开始（包括start），到下标end结束（不包括end）。其他的方法也是，begin与end都是上述意思。

```java
// 返回当前的容量
int capacity()
// 将StringBuilder的子串复制到char数组，可自定义从何处开始
void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
// 返回第一次出现String的下标索引
int indexOf(String str)
// 同上，但指定从fromIndex下标处开始搜索
int indexOf(String str, int fromIndex)
// 同上，不过是反向的搜索
int lastIndexOf(String str)
int lastIndexOf(String str, int fromIndex)
// 指定index处的字符修改为ch
void setCharAt(int index, char ch)
// 设置容量
void setLength(int newLength)
```

这些方法用法都写在注释里了，indexOf是有着对应的lastIndexOf方法，可以理解为indexOf是从开始进行搜索，而lastIndexOf则是反向搜索。

利用getChars可以将其中的子串复制到char数组中。这里有4个参数，srcBegin、srcEnd明显是指定复制范围，dstBegin则是指定了char数组是从该下标开始复制。

# String、StringBuilder、StringBuffer三者比较

首先看一下这3个类的源码中的签名：

```java
public final class String
extends Object
implements Serializable, Comparable<String>, CharSequence

public final class StringBuilder
extends Object
implements Serializable, CharSequence

public final class StringBuffer
extends Object
implements Serializable, CharSequence
```

这说明了很大程度上三者的方法等是十分类似的，而实际上String的方法是要多于其他两者的。

值得注意的是：非常常用的compareTo，是只有String有的。其他两个类需要转换成String后才能比较。

而append方法却是String没有的，String只有一个```concat()```方法，来将一个字符串拼接到原串末尾。

但是在操作的速度上因为String是字符串常量，而StringBuffer与StringBuilder都是变量，所以性能要差上一截，String是速度处理最慢的。

而StringBuffer与StringBuilder最大区别在于线程安全上其他基本是一样的。API中也描述了：

> This class provides an API compatible with StringBuffer, but with no guarantee of synchronization. This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case).

大概意思就是：

这俩一样的，唯一区别在于StringBuilder不能保证同步（线程不安全）。

StringBuilder是StringBuffer在单线程情况下的兼容代替者。

两者都是一个缓冲区，在缓冲区上进行操作，区别在于线程安全，StringBuffer的大部分方法都能够使用synchronization来保证线程同步（如果了解多线程会很容易理解）。

这样，在多线程的情况下，StringBuffer是明显安全的，而StringBuilder就不行了。但是也是因为舍弃了安全性，在性能上StringBuilder是要优于StringBuffer的。

注意API描述的最后一句：在单线程的情况下是更优的，单线程下没有线程安全这种说法，那么StringBuilder性能好、跟StringBuffer一样，那么当然要用StringBuilder了。

# 简单比较总结

在竞赛上：请使用C/C++（= =）为啥要用Java。。。好吧，用StringBuilder，竞赛是没有多线程的。

小Demo、Java作业之类的：String。方便快捷，简单粗暴。

项目：视情况，一般的数据类型之类的当然是String，只有在缓冲区大量操作的情况再考虑其他两个。

单线程且需要在缓冲区上大量操作：StringBuilder。首先要确保效率明显要高于String才行，其次因为不考虑同步、效率高。

多线程且需要在缓冲区上大量操作：StringBuffer。要考虑同步了。