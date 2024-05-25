---
layout: post
title: "Java异常获取中try-catch-finally块执行顺序实验探究"
author: "iwts li"
date: 2019-03-27 17:32:20
categories: Java
header-style: text
tags:
  - Java
description: "Java try-catch-finally 块执行顺序"
---

最近看面试题，发现这个比较有意思，try-catch-finally块的执行顺序问题。

一般认为，finally最后执行，做收尾工作，无论try块是否捕获异常，最后finally都会工作。

但是这样还是比较笼统，如果没有catch，而是将异常抛出，让其他方法处理，那么是先进入其他方法还是先执行finally？如果try块中return了，那么finally还执行不执行？进一步，如果try、finally全部有return，那么执行是怎样的过程？

确实，异常这些还是最早学Java的时候学的，当时似乎也没考虑这么多。借此机会研究一下异常获取的顺序。

# 节省时间，直接结论

try->catch->finally按顺序执行，不管是否有异常，不管try中有什么操作，就算是return，也得往后稍稍，最后这个方法一定是要执行finally。

如果try中抛出异常，而异常是留给上层方法处理，那么在抛出后，仍然运行finally，然后再回溯到上层。

自然，如果try中有return——也算是回溯了，返回值会存在栈中等待，等finally运行之后再回溯。

而如果finally中有return，那么直接从finally中结束方法。

如果在方法中直接结束程序，即调用System.exit()方法，那么就直接结束了，此时finally是不执行的。

由此可以认为，特殊情况导致程序的退出是可能导致一些问题的。毕竟finally一般写的是关闭对象、资源的代码。

# 通过代码实验分析

先写了一个包含情况比较多的例子：

```java
public class Main{
    public static int rank;

    public static void solve1() throws Exception{
        try {
            System.out.println("solve 1 try,rank: "+rank++);
            throw new Exception("throw by solve 1");
        }finally {
            System.out.println("solve 1 finally,rank: "+rank++);
        }
    }

    public static void solve2(){
        try{
            System.out.println("solve 2 try,rank: "+rank++);
            solve1();
        }catch (Exception ex){
            System.out.println("catch exception : "+ex.getMessage()+",rank: "+rank++);
        }finally {
            System.out.println("solve 1 finally,rank: "+rank++);
        }
    }

    public static void main(String args[]) {
        rank = 1;
        solve2();
        System.out.println("over");
    }
}
```

rank是计数。可以看到，整体上是先调用solve2方法，在其中调用solve1，solve1抛出了一个异常，让solve2捕获处理。大家可以先自己猜一下。

下面是返回答案：

```text
solve 2 try,rank: 1
solve 1 try,rank: 2
solve 1 finally,rank: 3
catch exception : throw by solve 1,rank: 4
solve 1 finally,rank: 5
over
```

根据上面的结果可以分析：try-catch-finally执行顺序：

首先是try执行，如果发生异常，那么直接捕获异常，最后执行finally。

但是，如果抛出异常，例如在solve1方法中，throw了一个异常，那么不会立刻回溯到上一个方法，而是仍然执行finally。

通过solve1的执行，我们可以认为，finally之前的所有代码，正常执行，但是返回之类的，全部被“卡”了下来，只有在finally执行之后，才能继续执行。

这里就又有疑惑了，一般认为throw了一个异常，就算是回溯了，为什么finally仍然执行了？如果这个不够明显，那么再看这个代码：

```java
public class Main{
    public static int solve(){
        try{
            System.out.println("try");
            return 1;
        }finally {
            System.out.println("finally");
            return 2;
        }
    }

    public static void main(String args[]) {
        System.out.println(solve());
    }
}
```

返回值是多少？

```text
try
finally
2
```

try块都已经return了，最后为什么是返回的return2？并且try块确实是运行了。再改一下代码：

```java
public class Main{
    public static int solve(){
        int i = 1;
        try{
            System.out.println("try");
            return i++;
        }finally {
            System.out.println("finally");
            return i;
        }
    }

    public static void main(String args[]) {
        System.out.println(solve());
    }
}
```

注意，try块返回了i++，那么我们debug就能看出来return这句到底是执行还是没执行，那么有这样的图：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405240008218.png)

可以看到，return确实是执行的。

所以，认为finally是无论怎样一定在方法的最后结束前执行的。

搜了一些资料，是说finally会在方法结束之前执行，而之前所有的执行，包括return，全部都停留在栈中，而finally最终执行后才继续。所以这样也能解释，第一次代码本应该回溯的代码执行完finally后才回溯，return的时候也是等finally执行之后再执行。

或许“return的时候也是等finally执行之后再执行”这句话又引出了一个问题：finally究竟是直接运行完结束，还是运行完之后再回到原来return的地方？

这里我们可以把i++换成++i，结果就不截图了——finally就是最终执行，如果有return，直接从finally返回。

还有一种情况，直接结束程序会怎么样？

```java
public class Main{
    public static void solve(){
        try{
            System.out.println("try");
            System.exit(0);
        }finally {
            System.out.println("finally");
        }
    }

    public static void main(String args[]) {
        solve();
    }
}
```

结果：

```text
try
```

强制结束大过天。

由此，也可以认为特殊情况导致程序直接结束，不会执行finally。因为finally一般写的都是关闭对象、资源的代码，所以这些特殊情况导致的程序强制结束，可能会引发一些问题的。
