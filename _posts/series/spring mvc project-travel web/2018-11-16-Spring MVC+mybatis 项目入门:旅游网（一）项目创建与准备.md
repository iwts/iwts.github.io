---
layout: post
title: "Spring MVC+mybatis 项目入门:旅游网（一）项目创建与准备"
author: "iwts li"
date: 2018-11-16 11:30:41
categories: Java Web
header-style: text
tags:
  - Spring
  - Java Web
  - MyBatis
description: "Spring MVC+mybatis 项目入门"
---
​
# 项目的创建与IDE

博主的是idea，好长时间没用eclipse了，不管怎样，如果想写Java Web，eclipse必须用企业版。这个就需要自己找破解之类的了。

idea的话，如果是在校大学生，可以直接申请教育版，几天就能使用jetbrains全家桶了。GitHub学生开发包也包含。

应该没有猛士不用IDE吧？不同的IDE影响不是特别大，但是博主没有用过eclipse企业版，所以用企业版的同学需要自己琢磨一些东西了。

首先是项目创建。idea下还是比较简单的，因为Spring MVC的包直接给你下载好了，不过版本并不是最新版，截止目前，提供的是4.3。其实也够用了：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405260220245.png)

可以选择直接下载，也可以用本地的。然后一路next就ok了。

mybatis就先不说，等用的时候再说吧。

# 创建文件结构

博主直接截个成品的树图：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405260220149.png)

src文件夹下，是放所谓的Java代码的，而web文件夹下，标准的前端三套：css、images、js，这里创建是为了后面使用图片、css等静态资源。

但是为了保护视图，再WEB-INF里面创建了view文件夹，存放JSP视图。

# 需要使用的jar包

还是总结一下吧，要不后面写到哪里就加载还是太乱了，直接截图：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405260221967.png)

这些jar包，全部在WEB-INF/lib文件夹下，不想自己下载可以直接在GitHub上获取。

这些jar包都有确定的用途，后面也会讲到，这里简单说明一下：

1. Spring那两个不多说了。
2. jar包第1-4个，是hibernate validator，即数据校验使用的。实际上有些情况下只有hibernate-validator和validation-api这两个jar包就可以了。当然在某些情况下，可能需要更多，这里就需要根据实际情况分析了。
3. 第5个，就是mybatis的jar包，使用mybatis就这一个就可以了，当然也可直接去官网下。
4. 第6个，JDBC驱动，虽然我们没有使用JDBC，但是要用mybatis啊，驱动一定要有的。这个驱动根据自己MySQL的版本是不一样的，也是一个坑，注意看好自己的版本。可以带上自己的版本直接百度，搜一下驱动的版本号。
5. 第7个，JSTL的jar包，前端一定程度上使用了JSTL。
6. 最后两个，文件上传需要的。Spring MVC有两种文件上传，博主使用的需要apache的包。另一种不需要，这个等到时候再详细讲。

# 上一章

[Spring MVC+mybatis 项目入门:旅游网（零）前言以及代码下载](../Spring-MVC+mybatis-项目入门-旅游网-零-前言以及代码下载)

# 下一章

[Spring MVC+mybatis 项目入门:旅游网（二）dispatcher与controller与Spring MVC](../Spring-MVC+mybatis-项目入门-旅游网-二-dispatcher与controller与Spring-MVC)
