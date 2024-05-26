---
layout: post
title: "Spring MVC+mybatis项目入门:旅游网（五）登录与注销"
author: "iwts li"
date: 2018-11-19 18:43:38
categories: Java Web
header-style: text
tags:
  - Spring
  - Java Web
  - MyBatis
description: "Spring MVC+mybatis 项目入门"
---
​
# 利用session实现登录逻辑

登录逻辑还是比较简单的，后端调用数据库进行数据匹配，正确则登录，否则返回错误信息：这里可以详细一点，例如声明是账号不存在还是密码错误。

这里并没有实现数据加密，而实际上应该是加密的比较好。如果加密的话，可能会将账号与密码合并在一起，所以这样就无法判定就是是账号不存在还是密码错误。

而登录成功以后实际上就可以利用session，将数据信息装进session，这样，只要session有相应的值，整体逻辑就应该按照用户登录的情况考虑。

例如首页，我们在用户没有登录的时候，显示登录按钮，如果登录就显示用户的昵称。这部分可以利用JSTL实现。

# 数据库匹配验证账号密码

mybatis的使用上一章已经写过了，这里就不多说。只用直接select获取值就可以了。

这里可以将用户全部搜索出来然后一个一个匹配，也可以直接将账号当做key直接搜索。如果有结果就可以认为找到了匹配，可以验证密码了。当密码验证成功就可以写入session。下面是代码：

```java
@Controller
public class UserController {
    public static SqlSessionFactory sessionFactory;
    public static SqlSession sqlSession;
    public static UserMapper mapper;

    // 查找数据库是否已经有该用户，并返回
    public static User selectUserByAccount(String account){
        try{
            User ret = mapper.selectUserByAccount(account);
            return ret;
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }

    // mybatis初始化
    static{
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            sessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (Exception e) {
            e.printStackTrace();
        }
        sqlSession =  sessionFactory.openSession();
        mapper = sqlSession.getMapper(UserMapper.class);
    }

    // 登录
    @RequestMapping("login.action")
    public ModelAndView login(@ModelAttribute User user, Model model, HttpSession session){
        User selectUser = selectUserByAccount(user.getAccount());
        if(selectUser == null){
            model.addAttribute("loginWrong","该账号不存在");
            model.addAttribute("user",user);
            return new ModelAndView(ViewTool.LOGIN);
        }else{
            if(selectUser.getPasswd().compareTo(user.getPasswd()) != 0){
                model.addAttribute("loginWrong","密码错误");
                model.addAttribute("user",user);
                return new ModelAndView(ViewTool.LOGIN);
            }else{
                // 可以登录
                session.setAttribute("isLogin",true);
                session.setAttribute("user",selectUser);
                session.setAttribute("userName",selectUser.getUserName());
                // 登录后进行一次转发，防止重复提交
                return new ModelAndView("redirect:/loginRedirect");
            }
        }
    }
    // 登录重定向
    @RequestMapping("loginRedirect")
    public ModelAndView loginRedirect(){
        return new ModelAndView(ViewTool.INDEX);
    }
}
```

可以看到，我的是直接利用账号在数据库进行搜索。而mybatis返回的是一个User对象，而这个对象就是当时定义的resultMap。接下来就是正常的匹配逻辑了。

最后允许登录的话，就可以对session进行设置。将一些对象放进去就可以了。这样前端就可以利用这些值完成操作。当然最后也需要重定向。重复登录是没有意义的。

这里我删减了一段代码，在源码中可以看到利用转发跳转的情况，这部分等之后再说。

# Spring MVC中对于request、session等对象的处理

这里不太入门了，可以看到，我写的方法里面直接声明了HttpSession对象，其他的例如ServletRequest等也都可以直接获得。

如果不想了解太多，就当做这些“JSP内置对象”在Spring MVC中也可以直接调用。其本质仍然是因为前后端直接交互，因为存在请求，所以可以获得request，进而session、application等对象都是可以直接获得。

简而言之，需要session的时候，直接在方法参数列表调用HttpSession的声明，然后就可以直接调用了。

这里就将一些判定值以及User对象存进session里面供前端调用。

# 利用session实现前端优化

配置EL语句，前端也可以进行新的操作了。可以看下现在JSP页的代码：

```jsp
<%@ page import="me.iwts.bean.Tourism" %>
<%@ page import="java.util.List" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <meta charset="UTF-8">
</head>
<body>
    <p>首页测试</p>
    <a href="/getRegister.action">注册</a>
    <c:choose>
        <c:when test="${sessionScope.isLogin == true}">
            <a href="/getProfile.action">${sessionScope.userName}</a>
        </c:when>
        <c:otherwise>
            <a href="/getLogin.action">登录</a>
        </c:otherwise>
    </c:choose>
</body>
</html>
```

利用JSTL和EL语句，可以方便地实现逻辑：未登录则显示“登录”，登录过了则把用户名显示出来。

# 注销逻辑与实现

注销当然也是非常简单了，把数据从session中抹除即可。

这里我将注销写进了新的页面，这个页面是用户处理页面，可以查看行程、修改个人信息等。

然后这里可以处理注销操作。入口就是当已经登录以后，自己的用户名是一个a标签可以完成发出请求的作用。

```java
// 注销
@RequestMapping("logout")
public ModelAndView logout(HttpSession session){
    session.setAttribute("isLogin",false);
    session.removeAttribute("user");
    session.removeAttribute("userName");
    return new ModelAndView("redirect:/logoutRedirect");
}
// 注销重定向到首页
@RequestMapping("logoutRedirect")
public ModelAndView logoutRedirect(){
    return new ModelAndView(ViewTool.INDEX);
}
```

# 管理员登录与注销

实际上就是Admin类了，其实整体几乎与User类一样，代码也很少变化。只是需要多配置Admin的mybatis以及写关于Admin的逻辑。

这里就不多说了。只有JSP部分稍微多了一点内容。可一看到首页：

```jsp
<%@ page import="me.iwts.bean.Tourism" %>
<%@ page import="java.util.List" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <meta charset="UTF-8">
</head>
<body>
    <p>首页测试</p>
    <a href="/getRegister.action">注册</a>
    <c:choose>
        <c:when test="${sessionScope.isLogin == true}">
            <%--管理员入口和普通用户入口要分开--%>
            <c:choose>
                <c:when test="${sessionScope.isAdmin == true}">
                    <a href="/getAdminConsole.action">${sessionScope.account}</a>
                </c:when>
                <c:otherwise>
                    <a href="/getProfile.action">${sessionScope.userName}</a>
                </c:otherwise>
            </c:choose>
        </c:when>
        <c:otherwise>
            <a href="/getLogin.action">登录</a>
        </c:otherwise>
    </c:choose>
</body>
</html>
```

多了判定是管理员还是用户。

# 上一章

[Spring MVC+mybatis项目入门:旅游网（四）用户注册2-持久化](../Spring-MVC+mybatis项目入门-旅游网-四-用户注册2-持久化)

# 下一章

未完待续——缺哥哥里什么时候出黑暗剑22啊？