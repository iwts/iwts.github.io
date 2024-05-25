---
layout: post
title: "Dubbo Admin搭建时Maven package失败"
subtitle: "记 Dubbo-Admin 搭建过程中遇到的大坑，搭建中的Vue与npm相关前端部署问题"
author: "iwts li"
date: 2019-06-02 15:28:09
categories: Dubbo
header-style: text
tags:
  - Dubbo
  - Node.js
description: "Dubbo Admin 搭建中的Vue与npm相关前端部署问题"
---
​
# Dubbo-Admin 的选择以及开源地址

首先吐槽一下网上的东西好老啊。。大概是17年左右的博客？基本上都是在配Dubbo的时候顺便做了Dubbo-Admin，并且都起名叫“Dubbo环境搭建”，干，这让萌新很苦恼啊，明明Dubbo不需要搭建的，只是如果选择Zookeeper当注册中心的话需要配Zookeeper，而Dubbo本身没有配服务端这种说法，所谓的搭建，其实都是搭建Dubbo-Admin。而具体用不用Dubbo-Admin跟使用Dubbo没有关系，就算不搭这个，该用还是用。

似乎在之前，Dubbo-Admin是包含在Dubbo内的，有个dubbo-admin文件夹，所以在里面打包使用。但是现在已经独立分出去了，需要单独找，所以大部分关于Dubbo的文章其实是错误的，因为已经不存在这个文件夹了。

现在的地址可以直接在GitHub搜索Dubbo-Admin，或者去官方：[Dubbo-Admin](https://github.com/apache/dubbo-admin)。具体搭建的时候，中文文档写的非常清楚。可以参考。

# 在使用Maven打包的过程中提示失败

这个就是大坑了，因为Dubbo-Admin是监控工具，用Spring Boot开发，所以需要打包之后运行，并且因为官方在Spring Boot中已经配了Web环境，所以按照文档直接启动运行即可。所以直接git clone下来并且打包即可。

但是在打包的时候就会出现失败的情况。大概提示这样的错：

```text
Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.6:npm (npm run build...
```

具体没截完。然后Maven的INFO显示，在ui层打包失败。之后一直搜这个错误，其实还是对Maven用得不太熟悉了。

之前的信息就不截取了，大概意思是：Dubbo-Admin的前端需要npm之类的使用。

看文档的时候发现，Dubbo-Admin前端是使用Vue的，所以需要构建。然而对前端这些真的不熟啊。后来查的时候才想起来上面的报错——错的其实并不是Maven，而是npm这些。

后来一直往上翻Maven的信息，发现在ui层，操作是这样的：

1. 首先判定机器是否有npm，如果没有就去下载安装。

   a> 这里可能碰到第一个坑——国内环境下装不了node.js和npm。这个可以自己手动装。博主比较幸运没碰见这个。
2. 之后为了启动Vue（没学过啊，只能这样理解），需要在ui下使用npm的build操作。

所以，这段错误的过程其实是这样：Maven发现没有装npm，手动下载配置node.js和npm，然后在dubbo-admin-ui下调用npm run build命令进行构造。

了解了之后，就去ui下，手动运行这个命令。果然错误了：

```text
building for production...Killed
```

等了两分钟就出现这个。

这样也可以理解之前Maven的错误了，Maven只是调用了npm命令，而本身在npm有错的时候只能说运行命令错了，而不能输出具体的错。

所以再去找npm出现这样的错误。找到了解决方法：

[一个解决方案博客，已经404了](http://www.woshuone.com/article/422)

撸了一遍——完事了。

## 新的解决方案

上面的博客404了。

可以百度
```text
building for production...Killed
```
解决方案基本都是来源上面的连接：

```bash
sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo /sbin/mkswap /var/swap.1 sudo /sbin/swapon /var/swap.1
```

## 一点排查问题的经验

一般来说出现这样的问题，应该看一下具体失败日志，可以用命令：
```bash
mvn clean package -e
```
并且需要详细分析一下之前Maven的输出。
​
# 经验总结

日志是真的牛逼啊，以及各种输出信息。

任何工具的错误基本上不会出现玄学事件，大家都是老老实实工作的。主要在于需要一层一层向上分析错误。

例如这次，Maven构建失败，ok，向上看，具体哪里失败了，失败的原因是什么。主要是了解做了什么导致了失败。

例如这次，之前就发现错误的地点了，但是就一直认为是Maven的错，所以在搜索这个错误的原因。

应该透过表象看本质，就像Java抛出异常：java.lang.NullPointException，大家当然知道是空指针，你去搜“Java NullPointException”怎么可能搜到解决方案啊，得先知道具体是在哪里抛出了这个异常，代码的前后都写了什么对吧。

回头再看，Maven抛出了错误，说是npm run build出现了错误，那就需要看在哪里运行了npm run build，为什么要在这里用这个命令。

所以只有正确找到问题的切入点，而不是xjb复制粘贴错误日志才能快速解决问题。
