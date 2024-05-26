---
layout: post
title: "Spring MVC+mybatis 项目入门:旅游网（二）dispatcher与controller与Spring MVC"
author: "iwts li"
date: 2018-11-16 16:53:03
categories: Java Web
header-style: text
tags:
  - Spring
  - Java Web
  - MyBatis
description: "Spring MVC+mybatis 项目入门"
---
​
# MVC模式

既然在使用Spring MVC，那么当然要了解什么是MVC模式。以下来自百度百科：

> [MVC](https://baike.baidu.com/item/MVC)全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。

MVC模式的各种图网上也有很多了，这里就不说了，说明一下我理解的Spring MVC：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261038288.png)

假设这个是祖国手办拼装场。肥宅对前台小姐姐说：给我一个蕾姆！前台小姐姐说ok，回头跟手办安装员说，给我一个蕾姆！手办安装员说ok，然后就需要从仓库中拿到蕾姆的各个组件。

仓库管理员直接在仓库里面找，找到以后全部给手办安装员。手办安装员开始拼，拼好以后给前台小姐姐，前台小姐姐在给肥宅。

这就是简单的一个逻辑，而也使用了MVC的思想。肥宅只用掏钱就行了，而前台小姐姐只负责传话，跟递手办。手办安装员哪里都不用去，只用安装手办就可以了，而仓库管理员负责在仓库里面手机零件，然后再给手办安装员。

我们可以去理解，前台小姐姐就是V，或者说是浏览器，负责接收肥宅的需求并且反馈给手办安装员。手办安装员就是C，只负责吧手办拼出来。而M就是手办。刚开始就是一堆零件，在被C拼好以后运送到V，然后肥宅就能看见了。

如果需要扩展一下，例如，不同的手办安装员精通某种类型的手办，因为肥宅的要求很高的，需要高达类的，就必须由高达安装员安装，而其他安装员不能安装。那么就需要多个C，同样，产品多了，就代表M多了。而前台小姐姐数量是不变的，增加前台实际上是多线程。

这样，我们的MVC工厂就略显混乱，1个V，多个C，多个M。而不同的零件可能换放在不同的仓库里面。

此时，我们可以升级为Spring MVC了。

Spring MVC提供了dispatcher，中文是调度员。当然就是负责中间调度的人员。dispatcher放在前台小姐姐和手办安装员中间。而对于仓库的管理我们交给mybatis。获得了这样：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261039062.png)

前台小姐姐还是很轻松的，传话就行了，具体这些话传给谁，全部由dispatcher来管理，dispatcher可以将需求传递给正确的安装员。安装员只用告诉mybatis需要什么就行了，mybatis就会自动找到具体的位置来把零件给安装员。

这也是博主理解的Spring MVC。而这次项目也会基于此。

# dispatcher的启用

dispatcher是一个servlet。

在没有框架的时候，利用jsp+servlet+javabean来完成MVC模式，我们就需要自己编写dispatcher。

而现在Spring MVC已经封装好了dispatcher，我们直接使用就可以了。不过其本质还是servlet，所以我们应该在web.xml里面对其进行配置，声明我们要使用这个servlet。

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

学过servlet，这里就很能看懂了。

正因为如此，在标签```<servlet-mapping>```里面的```<url-pattern>```反而有些疑惑。为什么是“/”。

其实就是字面意思，所有的请求全部映射到dispatcher。

因为dispatcher也是前置控制器，就是所有的请求全部经过dispatcher，而且统一由dispatcher进行调度。所以，Spring MVC的核心就是dispatcher。

而dispatcher的具体功能，包括IoC等全部被封装好了，我们根本不用考虑这些具体的实现，用就完事了。

当然，关于IoC等内容太多了，文章就不多写，这个还是需要自己下去查，因为这些比较核心，也有点不太入门。

## dispatcher 配置

现在，我们有了dispatcher，在web.xml里面的声明，Spring MVC就能知道：我们使用了dispatcher，并且对于所有的映射统统交给dispatcher来管理。那么我们当然也需要对dispatcher进行一些设置，配置dispatcher。

dispatcher的配置文件也是xml文件，但是命名要求是固定的。web.xml中，```<servlet-name>```是什么，我们的xml文件命名就必须一致，例如有：

```xml
<servlet-name>abcde</servlet-name>
```

那么我们dispatcher配置文件命名应该为：abcde-servlet.xml。

这个一定要一致。idea在创建项目的时候已经默认给建好了，很爽啊。

因为dispatcher是Spring MVC相当核心的部分，所以网上很多人也称其为Spring MVC配置文件。所以理解这个名词指的是什么就行，就是指dispatcher配置文件。

里面具体配置什么，现在先不说，但是文件一定要创建好。

# 声明controller

controller也是非常重要的，算是整个项目的核心部分。在Spring MVC2.0之前，我们都需要继承Controller类，通过重写一部分方法来完成controller的创建。

这样其实非常麻烦，并且一个类只能实现一个请求。项目稍微大一点就非常冗杂。而现在，我们可以使用注解方式，来很简单地完成操作。此时，我们可以开始项目了。

首先先来完成首页吧。我们想要进入首页，在逻辑上应该如何完成？

假设我们有一个请求：getIndex.action。这表示我们想要获得一个首页。此时，我们需要写controller类：

```java
@Controller
public class ViewController {
    // 首页
    @RequestMapping("getIndex.action")
    public ModelAndView getIndex(){
        return new ModelAndView("/WEB-INF/view/index-test.jsp");
    }
}
```

我们对整个类的上面声明@Controller。这说明，这个类就是我们需要的controller，而其中的方法是针对具体的请求而进行具体的操作，用注解@RequestMapping()来表示，里面就是具体的操作。

可以看到，@ReuqestMapping()注解是注释了具体的方法，说明这个方法处理了对应的请求。而其返回值ModelAndView就是指模型和视图。

这个ModelAndView类可以自动封装视图和模型，并且返回到前端。这里我们new了一个ModelAndView类，而其内容，则是一个视图在服务器上的位置。这样我们就能返回这个视图了。

# dispatcher如何找到controller

上面，配置好了dispatcher，完成了controller。

问题是我们应该如何让dispatcher找到controller。仅仅是一个@Controller注解就可以了么？当然是不对的，这里简单介绍一下扫描器。

dispatcher扫描器，声明了一个域，dispatcher在寻找一些资源的时候，就去利用扫描器扫描。

例如dispatcher获取了一个请求，就需要利用扫描器先扫描一下，如果发现这个类有@Controller注解，ok，就再去里面搜索其方法，如果碰见@RequestMapping声明跟请求一样，就运行方法体。

当然，这里就理所当然需要在dispatcher里面配置，代码如下：

```xml
<context:component-scan base-package="me.iwts.controller" />
```

这样，就说明了去这个包下面扫描。

# 发起请求

现在，小姐姐跟dispatcher就都就位了，就差肥宅了。这个问题有点蛋疼，获取首页这个请求由谁来发出。

如果是一个真实的网站，我们需要访问其IP地址，然后就出来了首页。而tomcat的话，我们访问localhost:8080，就会默认跳转到规定的首页（这个需要自己配置tomcat的xml）默认就是WEB-INF下的index.jsp。

那么我们在这里面调用就好了啊：

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>$Title$</title>
    </head>
    <body>
        <jsp:forward page="/getIndex.action" />
    </body>
</html>
```

利用forward转发，发送一个请求，然后dispatcher捕获这个请求。然后依靠扫描器去controller包下面搜索，找到了具体的方法，然后就开始运行，同时返回了jsp页面。

# 利用view resolver来简化对视图的操作

如果按照上面的写法，在返回view的时候，写的是一大串字符串，很麻烦其实。而我们可以通过配置view resolver来简化这个操作。

这个是一个类，在Spring MVC规范中，称这些类为bean，我们现在需要这个bean，来完成对视图的处理。

我们将所有的视图全部放在/WEB-INF/view/文件夹下，而我们的视图全部是jsp代码。

而view resolver其实就是声明一次拼接，前缀是什么后缀是什么，处理以后，我们在代码里面就可以只写view的名字就好了。

而view resolver在dispatcher里面的配置如下：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/view/" />
    <property name="suffix" value=".jsp" />
</bean>
```

prefix就是指前缀，suffix就是指后缀，这样，controller里面我们就可化简代码：

```java
return new ModelAndView("index-test");
```

其实这样还是不好。我们需要视图的地方非常多，如果改一个文件的名字，就要改非常多。那么我们就可以写一个工具类，专门来控制这些视图的名字。例如ViewTool类：

```java
public class ViewTool {
    public static final String INDEX = "index-test";
    public static final String LOGIN = "login-test";
    public static final String PROFILE = "profile-test";
    public static final String REGISTER = "register-test";
    public static final String REGISTER_SUCCESS = "register_success-test";
    public static final String ADMIN_CONSOLE = "admin_console-test";
    public static final String DETAILS = "details-test";
    public static final String ADD_SUCCESS = "add_success-test";
    public static final String TOURISM_CONSOLE = "tourism_console-test";
    public static final String MY_ORDER = "my_order-test";
}
```

这些静态变量，声明了具体的视图名字，这样controller也能化简为：

```java
@Controller
public class ViewController {
    // 首页
    @RequestMapping("getIndex.action")
    public ModelAndView getIndex(){
        return new ModelAndView(ViewTool.INDEX);
    }
}
```

很爽，这样jsp的名字随便改，我们只用在ViewTool类里面改一次就好了。

# 上一章

​[Spring MVC+mybatis 项目入门:旅游网（一）项目创建与准备](../Spring-MVC+mybatis-项目入门-旅游网-一-项目创建与准备)

# 下一章

[Spring MVC+mybatis 项目入门:旅游网（三）用户注册](../Spring-MVC+mybatis-项目入门-旅游网-三-用户注册)
