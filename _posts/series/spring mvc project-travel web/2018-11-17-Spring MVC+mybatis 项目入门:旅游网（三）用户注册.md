---
layout: post
title: "Spring MVC+mybatis 项目入门:旅游网（三）用户注册"
subtitle: "控制反转以及Hibernate Validator数据验证"
author: "iwts li"
date: 2018-11-17 17:20:43
categories: Java Web
header-style: text
tags:
  - Spring
  - Java Web
  - MyBatis
description: "Spring MVC+mybatis 项目入门"
---
​
# 注册原理

其实很简单，前端页面显示一个表单，然后由dispatcher传递到controller，controller调用数据库验证，如果ok，那就写入数据库，同时返回注册成功的视图，否则可以返回注册页，或者是到一个错误页。

# 依赖注入与控制反转

这里提一下，在最早接触servlet的时候，应该有老师会说，Java的POJO应该只有属性与构造方法，除此之外对于每个属性必须写其对应的getter、setter方法。而这里就是为了依赖注入。

具体的理论可以百度，这里就简单说明一下构造注入与setter注入：

```java
// 构造注入
public class Test(){
    private B b;

    public Test(B b){
        this.b = b;
    }
}

// setter注入
public class Test(){
    private B b;

    public void setB(B b){
        this.b = b;
    }
}
```

为什么要使用依赖注入或者说控制反转？（实际上，两者是相同的，只是在不同的角度阐述了上述操作）这里应该有专门的文章论述了。篇幅有限，这里不再解答，但是推荐搞懂这两者再继续阅读，毕竟这个非常核心。否则就是只会用而不知道具体实现了。

现在给出jsp代码与controller的代码以及User类的bean：

```java
public class User {
    private String account;
    private String passwd;
    private String phone;
    private String email;
    private String userName;

    public User(){ }
    public User(String account,String passwd,String phone,String email,String userName){
        this.account = account;
        this.passwd = passwd;
        this.email = email;
        this.phone = phone;
        this.userName = userName;
    }

    // getter setter 忽略
}
```

```html
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <p>注册测试</p>
    <form:form modelAttribute="user" action="register.action" method="post">
        账号：<input type="text" name="account">
        <br />
        密码：<input type="password" name="passwd">
        <br />
        手机号：<input type="text" name="phone">
        <br />
        邮箱：<input type="email" name="email">
        <br />
        用户昵称：<input type="text" name="userName">
        <br />
        <input type="submit" name="submit" value="注册">
    </form:form>
</body>
</html>
```

这个```<form:form>```标签是Spring MVC的标签，请当做正常的```<form>```标签，为什么写这个标签？后面的优化部分会说到。

```java
@Controller
public class UserController {
    // 注册
    @RequestMapping("register.action")
    public ModelAndView register(@ModelAttribute User user, Model model){
        
    }
}
```

具体代码我没有写，这里仅仅演示了Spring MVC如何处理依赖注入的情况。

可以看到，form表单对应了User类的一部分，然后就直接action给提交了，并没有对表单输入的数据进行封装。

而在controller类里面，我们在形参列表却传递了一个User类。这个User类使用了注解@ModelAttribute。

其实这里在Spring MVC，完成了控制反转或者说依赖注入的操作。

我们表单仍然还是只传递了一堆值，而dispatcher在获取请求以后，就利用setter注入，帮助我们封装好了整个User类，然后是将这个User对象给传递到register方法里面的。

而只要我们的形参列表声明了需要这个对象，那么Spring MVC就能够给我们这个对象。这个过程，就是控制反转。

所以说，从controller的角度看，叫控制反转，从Spring MVC的角度看，叫依赖注入。

而不管怎样，我们都能够获取到这个对象，并且这个对象已经被封装成为了model，之后的操作就是数据持久化了（当然需要先进行验证）。

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261123097.png)

# Hibernate Validator后端数据校验

这里也是需要大篇幅讲解的部分。。。推荐百度搜一下，提醒一下，这里坑略多，自己搜的时候学得会好一点，就是可能有很多错，博主也是踩了很多坑。

Spring是没有数据校验的。但是数据校验是比较重要的一环。

可能比较多的同学学的是JavaScript校验，这个是在前端控制数据的正确性，例如格式问题。但是，如果某些不怀好意的同学恶意操作呢？例如直接使用http提交数据，这样就能绕过前端直接给后端传递数据。

当然安全问题远远没有这么低级，但是在现阶段，这样的问题应该被我们考虑，而更难的安全问题就以后在进行处理。所以，解决这个简单的安全问题就是进行后端的数据校验——无论你怎么传，只要我能在后端校验，就能防止恶意传递数据。

比较简单的方法就是直接处理——我们已经将对象封装好利用控制反转给获取到了，那么我们就能获取其各个属性的值，然后直接一顿操作就行了。

但是我们现在想要逼格高一点的，同时还想节省代码量，所以我们选择利用其它技术来实现这个功能。

上面也说了Spring是没有数据校验的。简而言之，Java只提供了一些规范，说，只要你能实现这个规范，就能进行数据校验了，而hibernate validator就是实现了这个规范。

那么我们就只用获取其jar包，然后一顿调用，就能利用其来实现数据校验的操作。hibernate validator是实现了两套规范的，我们下面讲的主要依据最新的规范，比较简单，也更强大。

也可以移步：[Spring MVC利用Hibernate Validator实现后端数据校验](../Spring-MVC利用Hibernate-Validator实现后端数据校验) 有更为详尽的解释

首先，jar包自然是需要的。hibernate validator所必须的jar包是2个：
1. hibernate-validator.jar
2. validation-api.jar

但是个人推荐多增加两个，可以杜绝大部分错误：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261126636.png)

但是如果还是有错的话，就只能看log了，具体缺什么jar包就去下载什么jar包。这些jar包都可以直接百度下载，或者在我的项目里面/lib/ext下查找。而具体怎么在IDE里面添加就不多说了。

## 约束注解

之后，我们需要对bean进行一次升级，就是添加注解。而这个注解，就是对某个属性进行约束，规定这个属性必须满足怎么样的条件，否则就会返回错误。先看一下bean的代码：

```java
public class User {
    @Size(min = 6,max = 16,message = "账号不能为空，位数要为6-16位")
    private String account;
    @Size(min = 6,max = 16,message = "密码不能为空，位数要为6-14位")
    private String passwd;
    @Size(min = 11,max = 11,message = "手机不能为空，手机号码格式错误")
    private String phone;
    @Email(message = "邮箱格式错误")
    private String email;
    @Size(min = 0,max = 10,message = "昵称不能大于10位")
    private String userName;

    public User(){ }
    public User(String account,String passwd,String phone,String email,String userName){
        this.account = account;
        this.passwd = passwd;
        this.email = email;
        this.phone = phone;
        this.userName = userName;
    }

    // getter setter 忽略
}
```

可以看到，每个属性上面对应的@Size、@Email等就是注解。不同的注解有不同的作用，这里提供一些图，是从以前的博客上截的：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261128280.png)

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261128865.png)

利用注解，就能比较方便地进行约束。

现在只是声明了约束，而如果违反这个约束会有什么操作这个是在controller里面执行的，但是我们需要告诉controller这里违反了约束，也就是需要提醒信息。

可以看到，我们在注解里面写了message属性，而里面的内容就是我们自定义的错误信息。

其实不加也行，如果我们不想让用户看到这个信息的话，默认情况下也会有错误信息，不过是英文的。但是我们选择让用户看到，这样能提醒他们你写错了。

怎么让他们看？这个我们留到最后说。

## 后端处理

那么现在，就看我们怎么在controller里面处理这个约束了，看一下controller里面的代码：

```java
@Controller
public class UserController {
    // 注册
    @RequestMapping("register.action")
    public ModelAndView register(@Valid @ModelAttribute User user, BindingResult bindingResult, Model model){
        if(bindingResult.hasErrors()){
            model.addAttribute("user",user);
            return new ModelAndView(ViewTool.REGISTER);
        }
    }
}
```

可以与上面代码进行一些比较，其实主要是参数部分有变化：

1. 对于传入的对象，需要用注解@Valid声明。
2. 增加一个BindingResult对象。

第一个操作，主要是声明，在进行依赖注入的时候，需要对这个类的属性进行数据验证，而验证方式就是根据其对应的注解。

而BindingResult对象，就是在进行数据验证的时候，如果有错误，就将其message给添加到BindingResult对象里面。而调用其hasErrors()方法，就能判定是否是有错误的。

而具体如何处理，这个就根据实际情况判定了。例如我们对于无所谓的数据，例如用户昵称。我们允许用户不写昵称，但是我们看论坛的话，发现这个昵称会默认是用户名。这就是我们处理的结果了，如果发现有用户昵称为空，我们就将用户名给赋值进去。

当然，我们这里的逻辑就是告诉用户：你错了，请重新输入。

所以可以看到，我们直接返回了一个视图，同时将user对象封装进model里面，和视图一起返回到注册页面，所以下面就是看前端如何处理了。

## 前端处理

现在，我们将model返回到了前端，同时视图也返回回来了，这里先上一个完整未删减的代码：

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <p>注册测试</p>
    <form:form modelAttribute="user" action="register.action" method="post">
        账号：<input type="text" name="account" value=${requestScope.user.account}>
        <form:errors path="account"></form:errors>
        <span>${accountError}</span>
        <br />
        密码：<input type="password" name="passwd">
        <form:errors path="passwd"></form:errors>
        <br />
        手机号：<input type="text" name="phone" value=${requestScope.user.phone}>
        <form:errors path="phone"></form:errors>
        <br />
        邮箱：<input type="email" name="email" value="${requestScope.user.email}">
        <form:errors path="email"></form:errors>
        <br />
        用户昵称：<input type="text" name="userName" value=${requestScope.user.userName}>
        <form:errors path="userName"></form:errors>
        <br />
        <input type="submit" name="submit" value="注册">
    </form:form>
</body>
</html>
```

同样，可以跟最早的jsp页面比较，看多了点什么东西。

首先，类似于${requestScope.user.account}这样的代码是EL表达式。这个是非常好用的，推荐大家先去看一下什么是EL表达式，然后再回头看这里的代码。只能说，用EL表达式很爽。

然后，下面就默认大家会一点EL表达式了。

首先可以看到，多的一部分是value值。这个部分是完成了记忆功能。例如刚开始注册表单是什么都没有的，而我们注册以后，如果有错误返回，会发现表单是我们上次提交的信息，除了密码。

这里就是利用value进行记忆功能，value的值就是EL表达式，而刚开始EL表达式是找不到user对象的，因为我们只有在model里面将user返回，才有这个对象，所以EL表达式的结果是空。

而如果第二次返回，那么就有user对象了，从而能够将上次输入的结果给显示在界面上。

这个不是最重要的，重要的是下面的标签：```<form:errors>```，这个标签能够显示hibernate validator捕获的错误数据。并且将其message给显示出来。

而path属性就指定了，其显示哪一个属性出现的错误。

请注意：想要使用这个标签，那么就必须使用```<form:form>```标签，这也是我放弃```<form>```标签的原因。

所以，如果你想要让用户看到哪里错了，就需要在message属性写想让用户看到的信息，如果不想，就可以使用默认message了。给一个效果图吧：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261132809.png)

# 上一章

[Spring MVC+mybatis 项目入门:旅游网（二）dispatcher与controller与Spring MVC](../Spring-MVC+mybatis-项目入门-旅游网-二-dispatcher与controller与Spring-MVC)

# 下一章

[Spring MVC+mybatis项目入门:旅游网（四）用户注册2-持久化](../Spring-MVC+mybatis项目入门-旅游网-四-用户注册2-持久化)
