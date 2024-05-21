---
layout: post
title: "Spring MVC利用Hibernate Validator实现后端数据校验"
author: "iwts li"
date: 2018-11-06 20:12:11
categories: Spring
header-style: text
tags:
  - Spring
  - Hibernate
description: "Spring MVC利用Hibernate Validator实现后端数据校验"
---

吐槽一下，网上坑好多啊！不过采坑才能学习，写bug能力-1。

# JSR 303、JSR 349与Bean Validator

笼统来说，就是Java规定了一套关于验证器的API，规范先后发了两版，就是所谓的JSR 303与JSR 349。

然后提出了基于规范的实现：Bean Validator。

JSR 303是最早的，其对应了Bean Validator 1.0的版本，比较菜，然后后来扩展了JSR 349，提出了依赖注入、注解等内容，称为了Bean Validator。

关于Bean Validator，这并不是一项技术，也算是一种规范，需要对其实现。而博文使用的Hibernate Validator就是其中一种。

因为Spring MVC本身不提供validator的，所以我们需要其他的代替。下面是Wikipedia的一些内容：

> ​Java Bean Validation ([JSR 303](https://jcp.org/en/jsr/detail?id=303)) originated as a [framework](https://en.wikipedia.org/wiki/Framework_(computer_science)) that was approved by the [JCP](https://en.wikipedia.org/wiki/Java_Community_Process) as of 16 November 2009 and accepted as part of the [Java EE](https://en.wikipedia.org/wiki/Java_EE) 6 specification. The [Hibernate](https://en.wikipedia.org/wiki/Hibernate_(Java)) team provides with Hibernate Validator the [reference implementation](https://en.wikipedia.org/wiki/Reference_implementation) of Bean Validation and also created the Bean Validation [TCK](https://en.wikipedia.org/wiki/Technology_Compatibility_Kit) any implementation of JSR 303 needs to pass.

大概翻译一下，就是Java Bean的数据验证（JSR 303），起源于2009今年批准的框架，并成为Java EE 6的一部分。而hibernate团队提供了其一种实现。所以Hibernate Validator其实与SSH的那个H：Hibernate，并不是其子集。（可能吧，并没有接触过hibernate）。

# ​Hibernate Validator

上面也说过了其由来，这里简单说明一下运行过程。

JSR 349提出了注解约束，所以一般使用注解了。除去在项目中的配置，我们可以在Java Bean中的属性上使用注解来进行数据约束。

在Spring MVC中，利用IoC处理对象的时候就会触发数据验证。例如提交表单的时候，dispatcher接收的时候，就会根据约束来判定数据是否正确。

如果有不正确的，就会根据声明的message，将其添加到Error中，Spring MVC中一般使用BlindingResult对象来获取，然后就能对错误的数据进行处理了。

整体上，除了配置以外，还是比较简单的。

# 必须的一些jar包

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405212335032.png)

jar包是当时看的某本书，里面源码找的，可能会比较老，找新版本也可以。

然后这里有一点是比较坑的，网上大部分说的是2个包或者3个包——除了classmate的包。然后我这边是用前3个包的话报错了，很坑，然后看log大概是这样的错误：

```text
java.lang.NoClassDefFoundError: com/fasterxml/classmate/Filter
```

其实这里当时没截图，错误很多，但是比较重要的就是这个，发现是少这个包了。上网搜了一下，找了这个0.8的包，貌似最新的是1.0（不过好长时间不更了）。

然后在idea的话，记得在Artifacts里面添加，否则直接加jar包并没有什么用。这里我提供一下下载吧，百度云盘，挂了可以评论（大概能补）

> 链接：https://pan.baidu.com/s/1tsDsoHmM8xjA0MGy3CzWTQ 
提取码：qea7

## 2024年review

鉴定为当时比较sb，不管是当时还是放现在，直接Maven管理包是最合理的解决方案，请不要用上面自己装jar包的方案，过于old school了。

百度云盘也不给你补，给你补个蛋，蛋也不给你补 铁铁😁

# 注解约束Bean

JSR 349新增的，还是比较方便快捷的，先提供一个demo感受一下：

```java
public class User {
    @Size(min = 6,max = 16,message = "{user.account.size}")
    @NotNull(message = "{user.account.notNull}")
    private String account;
    @Size(min = 6,max = 16,message = "{user.passwd.size}")
    @NotNull(message = "{user.passwd.notNull}")
    private String passwd;
    @Size(min = 11,max = 11,message = "{user.phone.size}")
    @NotNull(message = "{user.phone.notNull}")
    private String phone;
    @Email(message = "{user.email.email}")
    private String email;
    @Max(value = 10,message = "{user.userName.max}")
    private String userName;
}
```

在属性上添加不同的注解，代表不同的约束，后面的message属性就表示了如果违反约束，应该报怎样的错误。message后面会详谈。

关于这些注解，就直接放找到的了。

## Bean Validation内置注解

|     注解名     | 简单介绍                             | 例子                            |
|:--------------:|:-------------------------------------|:-------------------------------|
| @AssertFalse   | boolean 类型必须为 false             |                                 |
| @AssertTrue    | boolean 类型必须为 true              |                                 |
| @Max           | 小于指定数                           | @Max(value = 150)               |
| @Min           | 大于指定数                           |                                 |
| @DecimalMax    | 小于指定数                           | @DecimalMax("2.2")              |
| @DecimalMin    | 大于指定小数                         | @DecimalMin("0.01")             |
| @Digits        | 指定整数位小数位的最大长度           | @Digits(integer = 5, fraction = 2) |
| @Future        | 必须是未来的一个日期                 |                                 |
| @Past          | 必须为过去的日期                     |                                 |
| @NotNull       | 不为 null                            |                                 |
| @Null          | 必须为 null                          |                                 |
| @Pattern       | 值必须匹配正则表达式                 | @Pattern(regexp = "\\\\d{3}")   |
| @Size          | 值长度必须在规定的范围之间           | @Size(min = 2, max = 100)       |

其实也没啥好说的，注意下@DecimalMax注解和@Max的区别吧。来自Stack Overflow，简单的区别，@DecimalMax接收的是一个字符串，所以支持例如大数类的匹配，而@Max则不能。其他情况例如double等类型，实际上两者是都能完成的。

## Hibernate Validation扩展注解

|       注解名       | 简单介绍                                | 例子                           |
|:------------------:|:----------------------------------------|:------------------------------|
| @Email             | 必须是 e-mail 地址                      |                               |
| @Length            | 字符串大小必须在指定的范围内            | @Length(min=2, max=10)        |
| @NotEmpty          | 字符串必须非空                          |                               |
| @Range             | 元素必须在合适的范围内                  | @Range(min=1, max=100)        |
| @NotBlank          | 字符串必须非空                          |                               |
| @CreditCardNumber  | 字符串必须通过 Luhn 校验算法（例如银行卡）|                               |

其实有些解释也是奇奇怪怪的，不过用得不太多= =有机会再找找具体用法和解释吧。

# message信息

一般我们是利用注解标志属性，那么当属性出问题的时候我们需要返回错误信息，而这些不同的错误匹配的错误信息需要自己设定，不管怎样都要在注解上写message属性，message属性里面就写返回的错误信息，这些会被BindingResult对象给获取。

其使用可以参考最上面的demo，或者看这里：

```java
public class User{
    @Email(message = "email error")
    String email1;
    @Email(message = "{user.email.error}")
    String email2;
}
```

这里用了两种写法，第一种，直接是一个String，第二种是利用properties文件的情况，下面会详细谈。

## properties文件

<p style="color: red;">这里有一点错，再研究一下，无法获取properties文件的映射，很神秘啊</p>

我们可以使用properties文件，将这些错误信息作为映射给表达出来。

关于properties文件，可能有同学学Java SE的时候就碰到过了，但是这里还是比较方便的，只用写properties文件就可以了，不像SE那样还得用反射获取数据源啥的。

下面提供一下简单入门，想详细了解的再搜搜相关内容吧。

properties文件，大概可以理解为像XML那样的配置文件。但是比较方便，就像Map一样是用key、value的键值对完成了映射关系。后缀名为.properties，内容及其简单，写就完事了，例如：

```properties
NAME=iwts
AGE=18
GIRLFRIENDS=NONE
```

这样就完成了一系列映射，写法是按照Java SE的，key大写了。注意等号前后不要有空格。

放的位置，一般在项目中是在src文件夹下。这样在使用相对路径找的时候，直接就是“/”。在src/properties下也行。在Java SE中怎么使用就不多说了。

在Hibernate Validator中使用的话，内容还是这样写，不过key是指具体的错误，value则是错误信息。

key的命名一般按照原则：
```text
对象名.属性名.错误类型
```
这样也会比较好找，例如最上面demo的对应properties文件：

```properties
user.account.size=用户名长度错误，请保持6-16位之间
user.account.notNull=账号不能为空
user.passwd.size=密码长度错误，请保持在6-16位之间
user.passwd.notNull=密码不能为空
user.phone.size=请输入正确的手机号
user.phone.notNull=请输入手机号
user.email.email=请输入正确的邮箱
user.userName.max=名字长度不能超过10位
```

然后扔进src文件夹下就行了，有地方放就ok。

### 是否使用properties文件？

博主也没啥经验，就不带节奏了，说一下自己的理解：快速开发或者写小项目（大学狗标准大X期末项目实训之类的），可以不要properties文件，直接写就完事了，修改也比较好找。

但是对于大一点的话这样写比较规范，各种修改都可以，反正对数据源是没有影响的。

# dispatcher上配置validator

就是在dispatcher对应的XML配置文件上配置validator，看网上有人也说是Spring MVC的配置文件，博主还是习惯说dispatcher了。

首先，不管怎样一定要有的是这些：

```xml
<mvc:annotation-driven validator="validator"/>

<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
        <property name="providerClass" value="org.hibernate.validator.HibernateValidator"></property>
</bean>

```

注册了validator的驱动，注意上面是将下面的validator的bean给注册进去的。所以```<mvc:annotation-driven>```标签的validator属性要跟下面bean的id相对应。

接下来就有点不同了。上面在谈message的时候，可以选择是否使用properties文件。这里根据是否选用配置是不一样的。或者说，如果不使用properties文件的话，上面就ok了，下面的都不用看了。

如果使用properties文件的话，总得声明映射关系对吧，怎么映射到这个资源文件。所以需要有下面的配置文件：

```xml
<!-- 驱动 -->
<mvc:annotation-driven validator="validator"/>

<!-- validator基本配置 -->
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"></property>
    <!-- 映射资源文件 -->
    <property name="validationMessageSource" ref="validationMessageSource"></property>
</bean>

<!-- properties文件 -->
<bean id="validationMessageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basenames">
        <list>
            <value>properties.user</value>
        </list>
    </property>
    <property name="fileEncodings" value="utf8"></property>
    <property name="cacheSeconds" value="120"></property>
</bean>
```

可以看到，主要是validator的bean里面多了一个属性，其ref指向了properties文件的bean。所以ref属性要与properties的bean的id一致。properties的bean里面的list里面value声明了其位置。只能用相对路径了，所以说properties文件应该放在src下面。

# 如何使用

其实上面配置完成基本就完事了，使用上，bean里面当然要写注解，剩下的在controller里面的代码需要一些变化，例如这个demo里面：

```java
@RequestMapping("register.action")
public ModelAndView register(@Valid @ModelAttribute User user, BindingResult bindingResult, Model model){
    if(bindingResult.hasErrors()){
        ArrayList<String> errorList = new ArrayList<>();
        System.out.println("have error");
        return new ModelAndView("register_error-test");
    }
    System.out.println("ok");
    return new ModelAndView("register_success-test");
}
```

主要的是形参列表，可以看到对于user对象而言，多了一个@Valid注解。我们需要这个注解来声明，依赖注入的时候顺便来一波数据校验。

而后面的BindingResult对象，在数据校验中，就是收集message，下面代码可以进行判定。

也可以使用List进行保存，例如添加到model里面返回给视图啊之类的。总之剩下的部分就看怎么玩了。

# 一个测试

其实看懂了就可以自己试一试，视图的话随便写一个jsp或者html有个form标签就行了。

controller的话如果搞不明白dispatcher（至少知道怎么用）的话还是推荐先搞明白controller之类的东西。上面的demo实际上就是一个小测试的合集了，可以复制代码试试看。不过自己写一个bean会更方便吧。

# 一个简单的使用

虽然上面配置之类的都完成了，但是应该怎么用呢？这里简单给出一个代码，大概实现的内容就是前端表格，输入以后后端利用Hibernate Validator进行数据校验，然后如果正确就不多说了，错误就跳回原来页面，并且显示message。大概是这样子：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405212351954.png)

很简陋啊，就这样吧 ，重点是后端的逻辑对吧。然后dispatcher配置文件就不放了，也没啥新的内容。具体的代码，改一改包名应该是能跑起来的（大概），其实不难，推荐自己手写一下吧

首先是视图，这里使用了JSP，代码如下：

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
        账号：<input type="text" name="account" value=${user.account}>
        <form:errors path="account"></form:errors>
        <br />
        密码：<input type="password" name="passwd">
        <form:errors path="passwd"></form:errors>
        <br />
        手机号：<input type="text" name="phone" value=${user.phone}>
        <form:errors path="phone"></form:errors>
        <br />
        邮箱：<input type="email" name="email" value="${user.email}">
        <form:errors path="email"></form:errors>
        <br />
        用户昵称：<input type="text" name="userName" value=${user.userName}>
        <form:errors path="userName"></form:errors>
        <br />
        <input type="submit" name="submit" value="注册">
    </form:form>
</body>
</html>
```

这里使用了表单标签库，比较方便的技术，推荐学习一下，引用之类的也比较快捷。

重点在于input的value。

可以看到，value使用了EL表达式。这样，如果有错误返回页面的时候，可以获取之前提交的数据。这样，如果错误的话，就会保留用户之前输入的数据。```<form:errors>```标签就是有错误的时候，输入框右边的提示。这里使用path属性获取了对应的bean的属性。那么当有错误的时候，就会显示设定的message。

bean也给看一下吧：

```java
public class User {
    //账号、密码、手机号、邮箱（可选）、用户名（可选，不选则用账号代替）、是否为会员、会员截止日期
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
    private boolean vip;
    private Date vipDeadLine;

    // 全部的getter/setter 方法
}
```

没啥好说的，getter、setter方法而已，注意上面是写了注解约束的。

最后是controller类：

```java
@Controller
public class UserController {

    // 注册
    @RequestMapping("register.action")
    public ModelAndView register(@Valid @ModelAttribute User user, BindingResult bindingResult, Model model){
        if(bindingResult.hasErrors()){
            model.addAttribute("user",user);
            return new ModelAndView("register-test");
        }
        return new ModelAndView("register_success-test");
    }
}
```

重点在于```register()```方法的形参列表，利用@Valid注解注释bean，要使用BindingResult对象获取错误信息。

然后网上也有Errors对象的写法，并没有使。返回的视图，因为我在dispatcher写了过滤器，大家如果直接copy的话推荐返回值改成String，然后直接返回自己的view的名字吧。 

# Hibernate Validator分组校验

留坑，之前看到的，还没仔细看。
