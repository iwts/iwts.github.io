---
layout: post
title: "JQuery Mobile默认Ajax异步提交导致URL不跳转问题"
author: "iwts li"
date: 2018-12-24 17:49:28
categories: JavaScript
header-style: text
tags:
  - JavaScript
  - Ajax
  - JSP
description: "JQuery Mobile默认Ajax异步提交导致URL不跳转问题以及解决方案"
---

最近隔壁实验室把后台交给我们来写了，要求前端用jquery mobile，因为只想在手机上跑。

期末了无暇顾及，瞎写的时候就出现了这个大坑= =。

例如注册之类的有数据提交的情况，并且还跟数据库大量交互。为了安全，博主写的时候写了重定向，这样在跳转的时候不会出现重复提交表单或者重复对数据库操作的问题，然后这前端框架就炸了：跳转的时候URL没变化！

网上搜了一波，大概是说默认jquery mobile是使用ajax异步提交的，所以URL是不会变化的（感觉自己还挺幸运的，网上有各种因为URL不跳转导致的路径问题，感觉这种bug改起来很坑，毕竟没啥事也不会看URL）。

此时需要需要调整不使用ajax异步提交。以一个a标签为例子：

```html
<a href="/student/register" data-ajax="false">Register</a>
```

需要在后面声明不使用ajax异步。

因为作用是在提交的时候不使用异步，那么扩展一下，在使用表单提交的时候这样是错误的：

```html
<form action="/student/register" method="post">
    <input type="submit" data-ajax="false" value="Register">
</form>
```

提交很明显是要写在```<form>```标签上的：

```html
<form action="/student/register" method="post" data-ajax="false">
    <input type="submit" value="Register">
</form>
```
