---
layout: post
title: "Spring MVC+mybatis项目入门:旅游网（四）用户注册2-持久化"
subtitle: "mybatis的配置与使用以及Spring MVC重定向"
author: "iwts li"
date: 2018-11-18 15:46:22
categories: Java Web
header-style: text
tags:
  - Spring
  - Java Web
  - MyBatis
description: "Spring MVC+mybatis 项目入门"
---
​
# MyBatis

> MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。
> MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJO映射成数据库中的记录。

以上来自百度百科。

其实不用mybatis的话，JDBC也能完成注册的操作。其实很简单，就是先找数据库是否有重复，如果没有重复将数据写入数据库即可。

# MyBatis进行数据库操作的过程

每个基于MyBatis的应用都是以一个SqlSessionFactory的实例为中心的。看名字可以猜出来设计模式是工厂方法，所以该实例应该是由工厂产生。也就是类SqlSessionFactoryBuilder。

SqlSessionFactoryBuilder则是从配置文件中配置的Configuration的实例构造出来的。借此配置文件，接下来就能获得SqlSessionFactory对象。

mybatis将所有的SQL语句全部转化成为了映射，这些映射在专门的xml配置文件中写。而对于每个POJO类，其下总共有两个文件：
1. Java接口
2. xml文件

而我们将insert、delete等方法写进接口里面，声明形参列表、返回值等等。

而xml文件就配置对于接口中的方法，映射了什么SQL语句对数据库进行操作。

sqlSession利用SqlSessionFactory对象来获得，这个对象可以获得上面的具体映射mapper对象，而这个对象可以调用接口中的方法，从而间接使用SQL语句对数据库进行操作。大概有图：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405261138708.png)

实际上就是：写一个mapper接口，里面的方法对应了mapper配置文件里面的具体SQL语句。利用SqlSession可以生成mapper对象，然后直接调用mapper对象的方法就可以了。

mybatis的具体原理不太入门，推荐去看官方文档，有中文版：

[mybatis – MyBatis 3 | 简介](https://mybatis.org/mybatis-3/zh_CN/index.html)

# mybatis基本配置——基于MySQL

一般，我们在src文件夹下创建mybatis的配置文件，可以起名为mybatis-config.xml。其内容基本上是固定的：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <!-- 声明什么方式连接，MySQL就是JDBC了 -->
            <transactionManager type="JDBC"/>
            <!-- JDBC配置 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <!-- ？后面是声明了编码等，防止数据库内容乱码 -->
                <property name="url" value="jdbc:mysql://localhost:3306/你的数据库名?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="数据库账号"/>
                <property name="password" value="密码"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 映射包位置 -->
    <mappers>
        <package name="me.iwts.mapper"/>
    </mappers>
</configuration>
```

里面已经写了一部分注释了，所以说，最好对JDBC有一点认识。

其实比较重要的就是加载JDBC驱动这个问题。根据本地数据库的不同，JDBC的驱动也是不一样的。这个需要视具体的MySQL版本而定。

如果你的是5.7，那么直接用我的驱动就可以了，否则就需要去专门再找合适的，并且装载到项目里面。

```<mappers>```标签，声明了利用这个配置文件，那些mapper的接口以及配置文件可以被找到。直接添加所在包就可以了，当然要是一个类一个类添加也行。

值得注意的是，mybatis并不需要在web.xml或者dispatcher.xml里面进行声明或者配置，他们之间是独立的。

# mapper接口与xml配置文件

上面也说道，接口里面声明方法，而配置文件里面写具体的SQL语句。

这里还是提醒一点：这部分内容非常庞大，推荐去看官方文档或者搜专门的教程。但是我的这篇博文只能说应该这样来写，而深层次的原理是没有的，只能说篇幅有限吧。

首先，接口部分一定要写好，先看一下UserMapper.java文件的代码：

```java
public interface UserMapper {
    int insertUser(User user) throws Exception;
    int updateUser(User user) throws Exception;
    int deleteUser(User user) throws Exception;
    User selectUserByAccount(String account) throws Exception;
    List<User> selectAllUser() throws Exception;
}
```

可以看到，例如insert、update等操作，是需要数据的，而我们直接传输了User类。

mybatis可以利用getter方法直接获得需要修改的值，而具体应该对应数据库表中的哪一项这个由配置文件决定。而返回值是int。

其实这里就算没有返回值了——这个int类型的变量是我们确定是否正常运行的，实际上很少需要这个返回值。

而后面的select方法就不同了，很明显我们需要获得数据库中的内容。

在使用JDBC的时候，我们是利用select后获得了一个结果集，而mybatis却允许你直接获得对象，自然，是使用setter方法进行依赖注入。

而具体数据库表中的哪个字段对应User类中的哪个属性，也是在配置文件里面完成的。

上面挖了坑啊，下面就先看一下配置文件UserMapper.xml：

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="me.iwts.mapper.UserMapper">
    <resultMap id="userResultMap" type="me.iwts.bean.User">
        <id property="account" column="account" javaType="String" />
        <result property="passwd" column="passwd" javaType="String" />
        <result property="phone" column="phone" javaType="String" />
        <result property="email" column="email" javaType="String" />
        <result property="userName" column="userName" javaType="String" />
    </resultMap>

    <insert id="insertUser" useGeneratedKeys="true" keyProperty="account">
        insert into user (account,passwd,phone,email,userName)
        values (#{account},#{passwd},#{phone},#{email},#{userName})
    </insert>

    <update id="updateUser" >
        update user
        set passwd=#{passwd},phone=#{phone},email=#{email},userName=#{userName}
        where account=#{account}
    </update>

    <delete id="deleteUser" parameterType="String">
        delete from user where account=#{account}
    </delete>

    <select id="selectUserByAccount" resultMap="userResultMap">
        select *
        from user
        where account=#{account}
    </select>

    <select id="selectAllUser" resultMap="userResultMap">
        select *
        from user
    </select>
</mapper>
```

可以看到，namespace完成了配置文件与接口的映射关系。

其实这里博主也没有深入了解，因为网上是有说法的：接口的名字与xml文件的名字必须相同。而实际上博主第一次确实被这个坑了一下，因为时间久远也忘记了当时这个namespace具体是怎么写的。所以这就有一个问题——到底是相同的名字关联了两者，还是这个namespace。这里博主对mybatis理解不深，就不瞎说了。以后懂了随缘补在这里吧。

首先，最上面的```<resultMap>```标签，标准的结果集了。而这也是上面说的：利用结果集设定了表中字段与POJO类的属性的对应关系。里面，```<result>```标签就是一个字段的映射。而```<id>```标签在一个结果集中只有一个，这个是mybatis的优化，跟我们的操作无关——或者说不影响我们的使用。

而下面的具体的标签，就对应了我们的具体操作。例如```<insert>```、```<select>```标签，是说明了我们需要什么样的操作。

而标签体里面写的就是具体的SQL语句。

id属性就与接口的方法名建立了映射关系，我们调用方法的时候就能找到具体的SQL语句。其他的属性，其实入门没必要太理解，本质上是mybatis的优化，可以提高效率，但是写不写对我们的结果是没有影响的。

但是对于select操作，一定要声明结果集的，这个结果集就是上面声明的```<resultMap>```，将id对应写上即可。

可以看到SQL语句里面有很多类似EL表达式的写法。这个是占位符的意思，意思我们这里需要某属性，而这个属性是外部数据。这个数据其实就是接口中声明的传入的形参。

# 获得mapper对象

其实这里是博主的问题：我的写法其实不太好，这也是后来才发现的——有同学写的时候将获取mapper对象的操作写进配置文件里面，这个其实跟Spring框架有点关系，大概就是在容器启动的时候就按照一定方式找到一定的配置文件来生成mapper，然后就可以在代码中调用了。

但是因为到项目几乎完成，博主才发现这个操作，所以也没有改动。只能说这样写了不少重复的代码。如果不想按照博主这样写可以直接去网上找一下其他写法。

博主是将这些写进了static静态块里面，这样，因为一个controller一般是对一个POJO进行各种操作的，所以在加载这个controller类的时候就能直接加载到JVM里面了，这样，很多controller方法都能使用这一个mapper。

不过还是跟上面一样：虽然化简了重复代码，但是不能杜绝重复代码。可以看下这段代码：

```java
@Controller
public class UserController {
    public static SqlSessionFactory sessionFactory;
    public static SqlSession sqlSession;
    public static UserMapper mapper;

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
}
```

这段代码其实没什么好说的，算是标准写法了，最终生成mapper对象。值得注意的是这些部分：

1. Reader加载mybatis配置文件。

   a> 博主最早就说了，将xml文件直接扔在src下，如果写在其他地方，请注意相对路径写正确。

2. mapper获取流程。
   
   a> 上面也说过了，利用SqlSession可以获得mapper，但是这个需要加载一个具体的mapper接口。
   
   b> 可以看到也是利用了Java的反射，所以每个mapper只能处理一个mapper映射，这里的类名在切换的时候也要修改。

# 注册逻辑

终于到了具体注册逻辑的写法了。

逻辑很简单，查数据库看是否重复——重复显示“该用户已注册”，否则注册成功并写入数据库。

也可以看到，mapper在获取以后，调用具体的数据库操作就是一行代码的事了，很方便。代码：

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

    // 注册
    @RequestMapping("register.action")
    public ModelAndView register(@Valid @ModelAttribute User user, BindingResult bindingResult, Model model){
        if(bindingResult.hasErrors()){
            model.addAttribute("user",user);
            return new ModelAndView(ViewTool.REGISTER);
        }

        // 验证是否已存在用户
        User selectUser = selectUserByAccount(user.getAccount());

        // 注册成功或者返回重新输入账号
        if(selectUser == null){
            // 处理一下bean
            if(user.getUserName() == null || user.getUserName().compareTo("") == 0){
                user.setUserName(user.getAccount());
            }
            try {
                mapper.insertUser(user);
                sqlSession.commit();
            } catch (Exception e) {
                e.printStackTrace();
                sqlSession.rollback();
            }
            // 注册转发，方式重复提交
            return new ModelAndView("redirect:/registerRedirect");
        }else{
            user.setAccount("");
            model.addAttribute("user",user);
            String accountError = "该账号已被注册";
            model.addAttribute("accountError",accountError);
            return new ModelAndView(ViewTool.REGISTER);
        }
    }
    // 注册重定向
    @RequestMapping("registerRedirect")
    public ModelAndView registerRedirect(){
        return new ModelAndView(ViewTool.REGISTER_SUCCESS);
    }
}
```

最上面的静态方法，是博主实写的时候，发现大量操作这个查找，所以就单独写出来了。因为User的controller类里面不仅仅有注册的代码还有其他很多，而对查找用的非常多。

具体的数据库操作，实际上就是mapper对象直接调用接口的方法就完成了。注意自己定义的形参列表与返回值就行了。

代码中实现了先查找是否已经存在该用户。如果存在，就对model里面返回一个新的数据，这里显示“该账号已被注册”，同时返回到注册页面。这里算是多了一行提示吧，那么在JSP你想要显示这个错误信息的地方利用EL语句显示即可：

```html
<span>${accountError}</span>
```

这样就ok。

如果没有问题，就是可以注册了，具体逻辑直接写就行了。

因为用户昵称是不强制要求的。所以，如果用户没有写这个昵称，就默认跟用户名一致。这里判定一下即可。然后直接insert插入。

这里可以看到与select的不同。这里其实是默认使用了事务处理，即只有调用commit()方法，才能正常操作，而如果有问题，可以rollback()回滚。这里其实没有太大意义，因为我们并没有写线程安全这部分内容，但是代码还得写，否则mybatis是不会调用SQL语句的。

# 重定向

重定向与转发的区别就不多说了，基础内容了，不懂的直接百度吧。

这里为什么单独写了重定向？可以看到return的时候不是返回一个视图，而是写成redirec:/后面跟了另一个请求。

这里大家可以试一下：如果不写重定向，直接返回到注册成功页面。然后按一下刷新，页面应该会出现提示框：是否重新提交表单。

试想一下，如果这个是银行转账页面，我们转了1000块以后，跳转到了转账成功页面。然后用户点了一下刷新——是否重新提交表单，然后顺手点了个确定。本质上转账的controller又处理了一次，也就是说又转账了1000块钱。

当然，我们这里就是重复注册了一次。这样是非常不安全的——银行的例子，用户绝对要去喷啊，转了1000块怎么少了2000。并且这里还不算用户误操作，而是你代码不够严谨。

然后再运行重定向的代码，发现刷新就刷新了，什么提示都没有。因为重定向以后，表单的数据已经完全没有了，自然地址也发生了变化（详情参考重定向与转发的区别）。所以之前的表单与当前页面已经没有关系了。 

重定向在Spring MVC中实现逻辑很简单，在需要返回视图的时候重定向到另外的请求，再其他方法里面再返回视图。等于说中间有一个跳板。具体的代码实现就像上面代码写的一样。

# 上一章

[Spring MVC+mybatis 项目入门:旅游网（三）用户注册](../Spring-MVC+mybatis-项目入门-旅游网-三-用户注册)

# 下一章

[Spring MVC+mybatis项目入门:旅游网（五）登录与注销](../Spring-MVC+mybatis项目入门-旅游网-五-登录与注销)
