---
title: JAVA应用内存泄漏排查记录
date: 2017-10-20 10:21:00
tags: 
	- Java
	- Linux
	- JVM
comments: true
---
>最近公司的一台服务器频繁报警，老大让我研究下代码出了什么问题，咋一看才知道代码是用大名鼎鼎的异步框架Vert.x写的，本文记录本菜鸟排查问题的辛酸过程，仅作为以后的一点经验参考。本服务器阿里云双核ECS实例，2G内存，CentOS7系统。

## 1.查看服务器硬盘内存状况
```
top #查看服务器内存和硬盘容量

```
![top](http://upload-images.jianshu.io/upload_images/5834071-261a02461a048ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看了下参数，以为这些挺重要的，其实发现CPU和Mem都很正常，内存看似要用完了，其实free+buffers+cached大概容量也有900M，完全够用。

## 2.查看服务器进程的堆栈
```
jcmd #查看进程，发现有两个java项目，一个是公司的web项目，另一个是报警的Vertx项目
jinfo [pid] 查看堆内存，发现一个项目的内存参数-Xms1450M，本项目采用的默认配置，暂时看不出分配多少内存
or ps aux | grep tomcat | grep -v grep  查看初始内存使用
jstack [pid] 看栈信息，发现很多歌vert.thread在等待，因为服务器相应缓慢，看了下Log，看到了NPE
```
初步分析，NullPointerException导致了服务器相应缓慢。
首先调查，为什么会NullPointerException,根据Log很容易找到原因，最后配合前端同事知道新版本的前端请求格式不对造成报错。
那么问题来了，报错为什么会导致应用相应缓慢。可能是连接没有释放，或者是持久层插入错误数据报错没有释放连接。

## 3 查看进程具体内存
```
jinfo [pid] 查看到初始化的配置 
```
看到Vertx最大堆内存300M,old区只有50M，这个我很纳闷，照理说默认配置新生区和old区比例是1:2起码也有200M,这样不会oom吗。
```
jmap -heap [pid] 查看堆内存各个分区大小,old区使用率90%以上，如果加大Old，如果还不行，肯定是内存泄漏了，就是对象没有释放。
jstat -gcutil [pid] 1s 发现最近发生了6次FGC old区依然降不下去
```
果然过了一天，log就出现了heap Out Of Memory.

## 4 尝试增加虚拟机内存分配
javaweb项目部署在tomcat下,系统为它分配了1450M的内存，打开tomcat/bin/catalina.sh，修改
```
JAVA_OPTS= -Xms1000M -Xmx1000M，这里保证初始分配内存和最大堆内存相同，放置系统调整内存空间产生的消耗。
```
同时给报警的Vertx项目增加内存，
```
java -Xms700M -Xmx700M -jar >>t.log 2>&1 & 将标准输出重定向
参数说明如下
-Xms 初始化对内存
-Xmx 最大堆内存
-Xss每个线程内存
-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64
-XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4
```
重启项目后，过一段时间继续查看
```
jmap -heap [pid] 发现堆内存Old区飙升，还是达到了90%
```
## 5 查看代码，检查内存泄漏位置
```
jstat -gcutil [pid] 60000  查看从程序启动到现在进行了多少FGC,查看下old区百分比
jmap -histo:live [pid] >f 打印堆内存存活对象信息
vi f
```
通过查看对象信息，从上往下发现有个对象EventItem被实例化了10w+次，这个类里面包含的数组，对象数组响应实例化次数也和其他不在同一数量级。确定是这个问题，回到代码中，我找到了初始化该对象的位置，应该是报错之后，request没有释放，request里面的private对象UploadRequest没有释放，导致错误的EventItem也无法释放。
于是我在出错的try.catch语句中直接将整个请求return,因为这些数据其实都是些无效的数据，直接return也可以节约不少开销。

## 后续观察
修改代码后，重新发布，已近观察3天，出错项目的heap的old区使用率终于降下来了，dump了对象信息也没有大量的EventItem，服务器运行终于正常，说明问题解决了。这个过程中经历了一些坑，比如错误的以为是另外一个项目影响造成，浪费了不少时间排查，不过学到不少东西。
jinfo/jmap/jstat这些命令是调优时候常用的，dump出文件查看对象信息，jconsole监控本地jvm运行状况，通过修改jvm参数调整服务响应时间等等。
代码没有贴出来，具体信息也没细说，因为涉及公司项目还是保密为好，文章就留着以后回顾下吧。