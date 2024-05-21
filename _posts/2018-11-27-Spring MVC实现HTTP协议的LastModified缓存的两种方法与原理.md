---
layout: post
title: "Spring MVC实现HTTP协议的LastModified缓存的两种方法与原理"
author: "iwts li"
date: 2018-11-27 23:22:06
categories: Spring
header-style: text
tags:
  - Spring
  - HTTP
description: "Spring MVC实现HTTP协议的LastModified缓存的两种方法与原理"
---

# 关于缓存方式的大概原理

首先，Spring MVC中，我们发出请求，就会返回view，而实际上这样就引出了一个问题：我的请求如果没有任何改变的话，也就是说上次请求获得的静态资源是没有变化的。

此时按照Spring MVC的机制，我们仍然返回了一套静态资源。显然这样增加了服务器的负载，以及网络带宽的使用。

可能Ajax能够解决这个问题——但是我们可以使用HTTP协议中的缓存机制，LastModified机制或者说“时间戳”缓存机制。

大概的原理就是：第一次发出一个请求，HTTP应该返回一个状态码200，表示正常返回view。而同时，在HTTP的响应头上增加一个“Last-Modified”属性，这标志了此次响应的系统时间。

而在获取响应头的数据以后，再次发送同样的请求的时候，会在HTTP请求头上多增加一个属性“IF-Modified-Since”。

而这个属性会引发服务器中的一次匹配：查看两次响应的时间戳。如果时间戳一致，服务器就认为这次请求与上次请求是一样的，HTTP返回状态码304，而此次返回的只有HTTP响应头，而静态资源就使用缓存的。

否则更新时间戳，同时返回静态资源。

# 使用LastModified的一种方法

第一种方法是网上比较常见的，一般搜到的也是这种写法。但是说实在的，感觉不太能用得着。

首先，如果想要使用LastModified，controller类就要先实现LastModified接口，并且重写```getLastModified()```方法。

那么我们用最基础的写controller的方法：继承AbstractController类或者实现Controller接口。所以感觉略微的不现实。当然下面有改良版。

先看一下Controller类：

```java
public class TestController implements Controller,LastModified{
    private long lastModified = System.currentTimeMillis();

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        System.out.println("start："+lastModified);
        return new ModelAndView("returnView");
    }

    @Override
    public long getLastModified(HttpServletRequest request) {
        if (lastModified == 0L) {
            System.out.println("此时应为0 ： "+lastModified);
            System.out.println("感觉没有意义");
            lastModified = System.currentTimeMillis();
        }
        System.out.println("lastModified: "+lastModified);
        return lastModified;
    }
}
```

这种比较基础的写法需要在dispatcher.xml中配置bean的，配置代码为：

```xml
<bean name="/test.action" class="me.iwts.controller.TestController" />
```

而发出请求以及返回的视图，简单写了两个jsp：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>$Title$</title>
    </head>
    <body>
        <form action="/test.action" method="get">
            <input type="submit" name="submit" value="submit">
        </form>
    </body>
</html>
```

返回的视图：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <p>return</p>
</body>
</html>
```

# 源码与API分析

这里，可能大家看得一头雾水：貌似在controller里面也没有干什么，这里我们看一下Spring MVC的文档，博主简单翻译了：

> DispatcherServlet 也允许处理器返回一个Servlet API规范中定义的last-modification-date。决定这个属性的方式很直接：DispatcherServlet 会先找到对应的controller处理映射，然后检测它是否实现了LastModified 接口。若是，则调用接口的long getLastModified(request) 方法，并将该返回值返回给客户端。

还是略微的迷，但是可以了解到是在dispatcher找到映射之后再处理的，那么我们可以看一下dispatcher的源码，然后找到了这一段代码：

```java
boolean isGet = "GET".equals(method);
if (isGet || "HEAD".equals(method)) {
	long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
	if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
		return;
	}
}
```

可以看到这里突出了一个比较奇特的属性：如果想要实现这样的缓存机制，method就必须设置为get或者head。这里也专门比较了一下。

当然，核心就在于这里：dispatcher如果发现是满足条件的话，是会调用getLastModified方法的，而很明显，这里我们的controller实现LastModified接口的话是会重写getLastModified方法的。而这个方法返回了一个时间戳。

而接下来发现我们调用了ServletWebRequest对象的```checkNotModified()```方法。

这里瞎分析一下：我们重写了方法以后，主要就是返回历史保留的时间戳，而在dispatcher中，当然已经得到了这次的请求头，所以就有了当前的时间戳。然后在dispatcher中进行对比。

而我们看一下Spring MVC API中```checkNotModified()```方法的表述，这里还是博主的简单翻译：

> 根据上次修改的时间戳，检查是否已修改请求资源

所以我们的猜测并没有错，具体的时间戳对比封装进了```checkNotModified()```方法，而根据返回的boolean类型，可以知道请求是否发生了改变。

这样以来：上面的代码基本就了解应该怎么使用了：

```java
public class TestController implements Controller,LastModified{
    private long lastModified = System.currentTimeMillis();

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        // 正常操作，具体返回不返回view不用我们管，dispatcher可能根本不执行这些代码，如果不要返回视图的话
        return new ModelAndView("returnView");
    }

    @Override
    public long getLastModified(HttpServletRequest request) {
        if (lastModified == 0L) {
        // 只用返回历史时间戳即可
        return lastModified;
    }
}
```

这样一来，使用还是比较方便的，核心封装进dispatcher，我们只用实现接口并返回数据就行了。

# LastModified缓存的第二种用法

上面的写法毕竟还是比较老的，现在大家都用注解了，@Controller或者@RestController注解controller类，而@RequestMapping注解具体的请求。而这样的话，仿佛不好使用这个方法了。

其实这里我是有点疑惑的：源码上看，dispatcher在找到具体请求以后，就调用checkNotModified进行对比，如果发现时间戳一致，直接return，从而中断当前请求。

理论上，仍然是controller类实现接口，重写方法就行了，其他没有任何变化。而实际上这样的想法是错误的。仿佛dispatcher每次匹配都认为是不同的请求。

博主找到的也都是上面的用法，但是看API的时候，```checkNotModified()```方法给出了一个demo，博主想：可不可以将dispatcher对这一段的处理自己写了。所以就有了这样的代码：

```java
@Controller
public class TestController implements LastModified{
    private long lastModified = System.currentTimeMillis();

    @RequestMapping("/test.action")
    public ModelAndView test(WebRequest webRequest,HttpServletRequest request){
        System.out.println("start");
        if(webRequest.checkNotModified(lastModified)){
            System.out.println("check : "+lastModified);
            return null;
        }
        System.out.println("no check : "+lastModified);
        return new ModelAndView("returnView");
    }

    @Override
    public long getLastModified(HttpServletRequest httpServletRequest) {
        return lastModified;
    }
}
```

仍然是实现了接口，但是在请求里面，额外多了WebRequest，等于说我主动调用```checkNotModified()```方法进行匹配。可以看到控制台的结果：

```text
start
no check : 1543331931401
start
check : 1543331931401
```

很明显，我们实现了缓存功能。
