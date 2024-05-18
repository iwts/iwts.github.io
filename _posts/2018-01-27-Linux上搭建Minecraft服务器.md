---
layout: post
title: "Linux 上搭建 Minecraft 服务器"
author: "iwts li"
date: 2018-01-27 19:24:41
categories: 运维
header-style: text
tags:
  - linux
  - minecraft
---

如果想和小伙伴们一起联机玩MC，那么完全可以购买一个云服务器后自己创建一个属于自己的世界。

并且实测人数少的情况下也不需要有多好的服务器，我就是用搬瓦工19.9刀一年的服务器搭建的。3、4个人玩还是很开心的。

下面就是步骤，小白也可以尝试着手动搭建一下。

# 系统要求
   
下面是度娘的MC系统要求：

| 人数 | CPU（cores） | 内存（GB） |
| :----: | :----: | :----: |
| 20-40 | 2 | 2 |
| 30-60 | 2 | 3 |
| 60+ | 2 | 8 |

上面就是服务器端的需求，所以如果只是和小伙伴们一起玩，也并不需要多优秀的服务器。下面是我的配置：

| CPU（cores） | 内存（GB） |
| :----: | :----: |
| 1 | 0.5 |

Linux系统：CentOS 6.5 x86_64

# 安装Java

MC是用Java写的就不再赘述了，由于服务器端的MC是一个jar包，我们在配置之后通过运行jar包来开启服务器端，同时我们在PC上打开后通过IP地址即可搜索并进入服务器。

所以我们首先要先安装Java。一般来说默认安装的是有Java 8的，如果没有可以按照下面的方法来安装：

## Java 版本验证&安装

验证是否安装Java，如果安装就查看版本：
```bash
java -version
```

下面是博主的Java版本：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/20240518173942.png)

如果不是Java 8或者没有安装，就用下面的方法安装：

```bash
sudo yum install java-1.8.0-openjdk
```

# 下载MC服务器端

这里有一个要求，就是你和你的小伙伴要拥有同样版本的MC。不同的版本对应着不同的服务器端，所以我们要下载正确的版本。

如何看MC版本呢，一般来说MC游戏左上角就写的有了，例如：Mineeraft 1.12。如果没有的同学可以启动MC游戏进入到开始界面即可，下面是博主的MC：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/20180127183515695)

可以看到左上角跟左下角都有版本号，博主的MC是1.12。

既然知道了版本号，那么用命令就直接下载对应版本的服务器端就可以了。需要根据版本号补全命令格式：
```bash
sudo wget https://s3.amazonaws.com/Minecraft.Download/versions/{版本号}/minecraft_server.{版本号}.jar

# case
sudo wget https://s3.amazonaws.com/Minecraft.Download/versions/1.12/minecraft_server.1.12.jar
```
这个命令会对应版本的mc服务器jar包。此时，minecraft_server.1.12.jar就躺在当前目录了。

# MC，启动！

上面的jar包下载完成之后，我们稍作准备就可以开启了！这里还是提醒一下对Java不熟悉的同学。我们知道可执行文件是.exe。双击直接运行，那么Java可以生成jar包，就是.jar文件，在Windows上也是可以双击运行的。

而我们下载的这个MC服务器端其实就是已经做好的jar包，所以我们在“双击”启动之前还是需要一些配置，以及用Linux的方法来启动。

首先，来看一下内存的使用，输入命令：
```bash
free -h
```
我们可以看到当前内存的使用情况，下面是我输入该命令后的显示：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405181748250.png)

根据free栏的内存数，我们可以确定该给MC服务器分配多大的内存（当然越多越好啦）。这个分配是java的执行命令，我们需要使用命令来运行MC服务器：
```bash
sudo java -Xms{初始启动分配内存} -Xmx{最大分配内存} -jar {jar包所在路径}/minecraft_server.{版本号}.jar nogui

# case
sudo java -Xms120m -Xmx160m -jar /root/mc/minecraft_server.1.12.jar nogui
```

关于路径是要绝对路径的，如果不懂可以在当前目录下使用```pwd```命令来获取绝对路径。

## 关于nogui参数

关于命令最后一个参数 nogui，是分开的单词 no gui，意思是不需要图形界面，直接用酷（zhuang）炫（b）的黑框显示，其实这样会大大减小内存的使用，如果你的Linux是有图形界面的，就可以不打这个 nogui。

## 配置

关于MC服务器端的配置，我们就需要修改这个文件了，同样在jar包所在目录下：server.propertices

可以看到，里面是对当前你创建的这个游戏的各种配置，像选择模式啦、世界生成的种子啦、是否有村民啦等等，就像PC创建世界时的各种选项一样。这里就不再介绍了，需要修改的同学自行百度。

# 启动成功

上面的命令输入完成，理论上我们就能启动了！这里注意，我们现在如果在Windows下，等于说已经进入到这个程序中了，所以不能再使用Linux命令，等待参数由0%一直到100%就启动完成啦！下面给出博主启动完成的后几行显示：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405181752829.png)

这样就启动成功了，不要有顾虑，直接启动PC端连接服务器进入MC吧！

## MC最常见启动失败-同意协议

我们在第一次运行完jar包后，无论是否运行成功，都能发现当前目录下多出了一堆文件，运行失败的时候其实就是配置出了一点问题。

我们在当前目录找一下文件：eula.txt。我们需要对这个文件进行一下小编辑，如果是学计算机的直接将eula改成true就可以了。

不太懂的需要注意，Linux的vim编辑操作，请按照步骤来：

1. 输入```vim eula.txt```
2. 输入```i```，进入vim的编辑模式
3. 上下左右键可以移动光标，找到这一行：```eula=false```
4. 将false改成true
5. <span style="color:red">重点！接下来一定按照这样操作！</span>

   1> 按下ESC键

   2> 输入：```:wq!```

   3> 按下回车键

修改成功。这样就等于同意了那个“最终用户许可协议”，反正你要玩就得同意。

# MC，关闭！

启动之后当前这个窗口就可以不用管啦。服务本身是一直在服务器上跑的，你只是remote连接上去，直接关闭窗口退出就行，除非你想直接kill掉服务。

# MC，Shell脚本启动！

我们如果一直使用上面那一句启动的话是不是非常麻烦！每次都要复制粘贴，那么我们可以写一个简单的Shell脚本，放在jar包所在目录，每次启动的时候直接启动脚本就能进入游戏了，完全不需要Shell编程基础，直接复制粘贴即可！（玩游戏就不要想太多！当然玩爽之后可以了解一下Shell编程）

新建文件：start.sh

代码：
```shell
#!/bin/sh

java -Xms120m -Xmx160m -jar /root/mc/minecraft_server.1.12.jar nogui;
```

上面代码需要替换成你自己的启动命令。

是不是非常简单呢！其实就是让脚本帮你搓启动命令，而你仅需要运行一下脚本即可：

```bash
bash start.sh
```

MC，启动！

# 关于PC连接问题

之所以放到最后是怕有同学上面的都完成之后发现不知道如何连接到服务器上，或者说根本不知道服务器的IP地址。这里就说一下Linux下如何查看IP地址：
```bash
ifconfig
```
就是如此简单粗暴，一般来说，我们看eth0网卡就行了。如果嫌麻烦，直接看类似格式：192.168.0.1差不多的就是IP地址啦，其实写着 inet addr 的，后面的就是IP地址。然后在PC端多人游戏中搜索服务器时将IP地址输入就可以了。

# 个人遇到的一些问题
## MC连接失败

这里是上面都启动成功之后，PC端也搜索到了服务器，但是就是连接失败，这样我们可以修改配置，先在jar包目录下找到文件server.propertices 并编辑。

将online-mode:true改为false，这个好像是跟正版有关。