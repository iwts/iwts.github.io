---
layout: post
title: "idea中Maven/Web项目出现 java.lang.ClassNotFoundException 异常的解决方法"
author: "iwts li"
date: 2018-12-09 09:48:11
categories: IDE
header-style: text
tags:
  - IDE
  - Spring
  - Java
description: "idea中Maven/Web项目出现 java.lang.ClassNotFoundException 异常的解决方法"
---

首先说明一下，java.lang.ClassNotFoundException异常是有通用的解决方法的。一般而言，都是因为缺少某个jar包。而且在IDE中出现颇多。后面会跟一个包名，说明这个包没有引用。

在IDE中，如果我们需要某个特殊的类，一般会提前引用jar包，就算不引用，IDE会直接飙红，提示没有这个类，所以既然能跑就肯定引用这个jar包了，所以出现这个错误的原因并不是没有引入jar包，而是IDE没有将这个jar包“放入正确的位置”。

所以我也不太懂上面这个情况怎么描述。简而言之，我们在入门学习Java Web的时候，例如引入了commons-io.jar，书上、教程之类的会让我们把这个jar包放入WEB-INF/lib下，因为这样项目会引用到。

而idea比较特殊，不管是maven引入依赖或者是手动在moduies里面添加，只是说明这个项目引用了jar包，而并没有放进所谓的“正确的地方”。

那么结果是，我们的代码没有问题，因为能够获取这个jar，而项目在跑的时候却根本找不到这个包，所以只能抛出异常 java.lang.ClassNotFoundException。

出现这个问题，大家可以看一下项目的输出位置，默认的是在idea目录下：
```text
out/artifacts
```
可以看到自己的项目，而里面WEB-INF文件夹下，一个classes，一个lib。

classes放的是字节码文件，而lib里面就是jar包，如果里面没有刚刚加入的jar包——那么这篇博文大概率能解决问题，否则就还是继续搜索吧。

如果是非maven项目，Project Structure（左上角File下，或者右上角搜索左边的文件夹）里面的Problems是会有提示的，提示这些包没有加入到Artifacts里面，点fix就能直接解决。

而maven项目就不会有提示了，需要在Artifacts->Output layout里面，将右边的jar包，双击，添加进去。例如下图：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405221121222.png)

之后大概率能解决问题，如果还不可以——推荐好好看一下log，从下往上看= =可能问题不是在于java.lang.ClassNotFoundException，或者是隐性调用了其他jar包，例如poi.jar如果不是final版的，是会需要poi-shemes.jar(好像是这个没啥印象了)的。