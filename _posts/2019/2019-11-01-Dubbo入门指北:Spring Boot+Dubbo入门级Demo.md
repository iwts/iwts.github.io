---
layout: post
title: "Dubbo入门指北:Spring Boot+Dubbo入门级Demo"
author: "iwts li"
date: 2019-11-01 16:59:06
categories: Dubbo
header-style: text
tags:
  - Dubbo
  - Spring Boot
description: "Spring Boot+Dubbo入门级Demo"
---
​
# Zookeeper

至少要装个单机环境，Dubbo需要Zookeeper环境。具体的配置这里不解释了。

最后在Zookeeper目录下的bin目录下对zkServer.sh使用start即可开启Zookeeper。记得开放端口，Zookeeper端口需要自己配，默认配2181。

# Java

前提条件了，不多解释装就好了。

关于Tomcat，本身Tomcat就是一个Web容器，如果你想跑出来项目，那么还是要装的。但是对于Spring Boot开发而言，内置的Tomcat就可以使用了，单独装一个Tomcat服务器主要是为了使用Dubbo Admin。

# Maven
现在都要用的吧。

# Dubbo Admin
如果有监控的需求可以单独搭建。

# Dubbo与RPC？

网上坑不少啊，感觉就是用Dubbo的大佬不屑于写这些东西，想搞的要么是各种复制粘贴，要么就是好多年前大佬自己摸索的，有点蛋疼。

Zookeeper+Dubbo，通过这个Demo也大概摸到了一点点门路。给一个简单的场景：A需要调用B的数据。而现在软件的体量越来越大，基本不存在一个系统直接扔tomcat跑这种。都是RPC/SOA这种，分成多个服务，而服务与服务之间调用。

简单的说，可以认为对两个Java项目，分别打两个jar包，一起运行之后，需要数据进行交互。

jar包之间交互可能不太好办了，那么换个思路，如果是两个war包部署到tomcat上呢？这样最简单的想法就是：第一个网站发起一个请求，请求第二个网站。这其实也是一种分布式解决方案，即通过HTTP进行数据传输。

而Dubbo则是通过开放API的形式，也就是比较标准的RPC思想了。

对于网络，我们就是认为请求直接发送过去了，地址啥的都对，肯定就能建立TCP连接进行传输。而整个网络其实是在协议之上的——HTTP协议、TCP/IP协议群，就是因为有这些协议，互联网才能建立起来，否则怎么简简单单就能搭建数据传输的桥梁？（这里极度简化了计算机网络）

而RPC也是想通过一种协议，能够统一管理多个jar包，让多个jar包之间能通过开放的API进行数据传输。当然RPC连接的是多个服务。但是RPC是基于网络的，也就是说，RPC提供了一套协议，让多个jar包之间能够进行连接，而具体的传输，是依靠网络。

可以这样认为，N多年前的邮局，基本上只能一个范围内传输，可以认为是一个国家吧。而跨越国的送信有很大的障碍，语言不通等等。这就可以认为是一个国家就是一个项目，项目与项目之间没有API能够互相调用。

而现在不一样了，RPC框架的思路就是当一个联合国老大，规定：A国只要遵守贴怎样的邮票，就能与B国进行送信服务。等于说，联合国制定了一套规则，能够正确的连接两个国家的邮箱。而计算机网络，就可以认为是送信人——专门开车、飞机，往返送信。

那么对于Dubbo而言，就是在各个国家的海关建立邮局，只要在我驻扎的国家，大家都可以用我的邮件用我的邮递员进行送信。

Dubbo制定了规则。而Dubbo本身只是制定了规则，并没有一套有效的监控——监控那个国家可以那个国家不可以，以及运输的疏通等等。以及如果有新国家想要加入Dubbo，那么总不能先跑到别的国家的Dubbo海关吧？而Dubbo就是一个商业公司，没那么大魄力。

所以这个工作就委托给联合国——Zookeeper。当然也有其他的可以当，但是Zookeeper会更多一点。

# Zookeeper的启动与Dubbo Admin的监控

基本能理解这俩是干啥了。

所以Zookeeper是必须的——因为联合国必须有，而Dubbo Admin只是方便Dubbo个人也能得到一点消息，不要也无所谓——Dubbo送东西拿钱就行了。对于用户，可以从双方获取监控，但是Dubbo Admin毕竟是网页版，比较爽。

# 统一的开放API

RPC就是通过统一规范了接口，才能正确进行匹配。而Dubbo的做法就是开放一个公共接口，或者说做一个声明——生产者提供了一个接口，消费者想用就可以直接调用。自然这个就需要专门写一个服务了。

这个比较特殊，也可以理解为一个项目。但是里面只是写了pojo、service接口等等。并且根本不需要启动。就是完全作为一个API规范使用的。

虽然是Maven项目，但是pom.xml也不需要写什么，设定一下Java版本之类的即可。自然什么框架也不用。所以就需要写一个pojo，一个service接口。

项目名字是dubbo-common。pojo一个Person类，service接口就声明了一个方法：

```java
public class People implements Serializable {
    private int id;
    private String name;
    // getter、setter
}

public interface PeopleService {
    People getPeople(People people);
}
```

所在包需要做好。

然后，用法是什么呢？就是打成jar包。正常的MVC是怎么面向接口编程的，就是写一个接口，然后具体有对接口方法的实现对吧。

其实RPC就是将MVC分开了，生产者有对service接口的实现，而消费者有service接口的实现类——直接调用。所以这个接口就被作为API放在中间。

那么生产者消费者都需要这个API，所以就打成jar包，然后两方直接引用就行了。同时，这样搞在例如调用接口中声明的pojo、方法等，也不会报错。

因为Maven，这一切就变得非常简单：

```bash
mvn clean package
mvn install
```

在有私库的情况下，直接上传到私服，然后生产者与消费者引用就行了：

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>dubbo-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

就跟正常引用一个包一样。

# 消费者

首先考虑消费者。

消费者首先是一个服务，自然需要tomcat等Web容器，也就是说，消费者就跟写一个正常的项目非常类似了——并且主要是接受请求的，所以还是非常上游的一个服务。也就是说需要写Controller类，接受请求返回数据。但是对数据的操作，就需要Dubbo开放的API了。

消费者不关心数据怎么出来的，只需要知道，我可以调用service接口中的方法，并且获得数据就行了。所以当前也不考虑生产者，就关心，正常怎么写代码。首先Maven依赖不解释，最后专门说一下依赖，生产者和消费者是样的。

因为是Spring Boot，那么就先说一下yml的配置：

```yaml
server.port=8080 # 服务器端口
dubbo.application.name=dubboconsumer # 消费者服务名
dubbo.scan.base-packages=com.example.controller # dubbo接口扫描范围
dubbo.registry.address=zookeeper://{ip}:2181 # zk注册中心配置
dubbo.consumer.check=false
```

最后一行是一个检查，这个留在最后说。

然后就是Controller了：

```java
@RestController
public class PeopleController {
    @Reference
    private PeopleService peopleService;

    @RequestMapping("/people/{name}")
    public People getPeople(@PathVariable("name") String name) {
        People people = new People();
        people.setName(name);
        return peopleService.getPeople(people);
    }
}
```

而这个PeopleService就是开放的API了，我们既然引入了common包，自然就可以引用这个方法。

而具体怎么得到生产者的数据？这个就交个Dubbo了，消费者并不关心数据是怎么实现的，是怎么运输过来的，只需要知道——我可以得到这个接口的实现类，并且调用方法。

# 生产者

与生产者相反——我不关心我的数据给谁，反正有人问我要，我就给，我不关心是Dubbo还是其他的服务。那么生产者需要Web容器么？需要——要不怎么得到请求呢。

可以理解为生产者就是饭店厨师，生产外卖，Dubbo就是外卖小哥，负责拿外卖然后送给消费者。厨师为什么要管外卖小哥是美团（Dubbo）还是饿了么（其他RPC框架），更不管消费者是不是漂亮小姐姐了，只关心做饭。但是你得有网接收订单对吧。

但是生产者用户是不必关心的，所以对于端口，随便来一个就行，但是需要web环境这个是必须的。其次，就是实现service接口的具体实现。所以yml配置如下：

```yaml
server.port=-1 # -1就是随便找一个没人用的端口
dubbo.protocol.port=-1 # Dubbo运行端口，同上
dubbo.protocol.name=dubbo
dubbo.application.name=dubboprovider
dubbo.scan.basePackages=com.example.provider.service
dubbo.registry.address=zookeeper://{ip}:2181
```

而专门有serivce接口的实现：

```java
@Service
public class PeopleServiceImpl implements PeopleService {
    @Override
    public People getPeople(People people) {
        people.setId(new Random().nextInt(10000));
        return people;
    }
}
```

这里注意一点，这个```@Service```不是Spring Boot的注解，而是Dubbo的：

```java
com.alibaba.dubbo.config.annotation.Service
```

因为Dubbo需要管理接口，知道common里面声明的接口最后被那个生产者实现了。

# 一些坑

先说一些坑吧，把上面代码给完善完善。

## Spring Boot与Dubbo版本问题

这个可能是网上文章比较久远的问题，首先——低版本没问题是一定错的！很多人也是xjb做，然后发现低版本恰好没啥问题，所以就有这个说法——因为Dubbo是不断在升级的，而基本上各种项目都是N手的，所以就会这么玄学，xjb写恰好能跑的很多，但是大概率自己这里都跑不了，改低版本恰好符合之前的写法，所以就ok。

所以新旧版本至少在基础使用上没什么问题的，只是对于配置方法上，yml、xml或者写@Confiuration配置类。这三种网上都有。

而对于Spring Boot的版本选择上，必须用Spring Boot 2.x之前的版本是不科学的，例如博主用的就是2.1.5的版本。

2.x之后不能用的主要原因是因为服务注册的问题，2.x之前，```@Reference```这个注解就够用了，再加上老版本的Dubbo和Spring配合，也能将服务注册上去（很玄学，反正我没弄好）。

但是现在都是在启动类上加注解，这样来注册，就能完美解决生产者没有注册进去服务（生产者启动，Dubbo Admin没有检测到服务）、消费者调用API接口实现类抛出null指针异常的错误。

具体是这两个注解：

```java
@EnableDubbo()
@DubboComponentScan()
```

```@DubboComponentScan()```里面可以写具体扫描的包，不过一般不写。

至于Dubbo版本，一般用Spring Boot，用Dubbo的start就行。比较难找，必须要找阿里巴巴的。并且需要zkclient的包，这样方便操作Zookeeper，所以这两个包必须的。

具体的Maven如下，是当前文章最新的，也可以直接去Maven上搜，就是阿里的Dubbo start整合包很难搜到。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>dubbo-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>dubbo-provider</name>
    <description>描述，xjb写</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.spring.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.11</version>
        </dependency>
        <!-- 这个就是引入的common包了 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 启动顺序

说必须先启动生产者后启动消费者比较蛋疼。。。主要还是因为不这样会出现null指针，直接报错之类的，主要是理由蛋疼，这样根本不符合场景，怎么可能必须要这样的顺序啊。

解决方法很简单，在消费者配置的时候：

```yaml
dubbo.consumer.check=false
```

# 启动

尾声。

首先Zookeeper是要启动的。如果配了Dubbo Admin，也启动看看。之后陆续启动生产者与消费者即可。

生产者启动后就可以被Dubbo Admin监控到了。然后----调用方法吧。

# Demo包

很遗憾，应该是在公司电脑忘记拷贝了，应该是没有了，不过这种入门demo还是比较好复刻的。

这就是不存Github的下场。。。
