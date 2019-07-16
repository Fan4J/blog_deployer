---
title: Charles进行Htpps抓包
date: 2017-10-10 09:04:14
tags: 
	- 系统架构

---
>今天在公司打开电脑发现没法登陆Https的网站，知乎百度等，github也上不了，后来查看了自己电脑的代理设置发现代理还开着，原来是蓝灯开机启动直接挂着代理，发现本地还在用Charles代理，研究了下，Charles代理相当于从本地的localhost:8888地址发送请求，如果遇到Http协议的网站，可以直接解析获取数据，遇到Https的网站，无法获取数据包，特此研究一下。

## 1.Charles抓包原理和设置

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5834071-b1a503f7f689ad8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这段话已经清晰的解释了Charles抓包SSL协议的原理，这里首先要回顾下SSL协议请求的原理

![Charles SSL Proxy](http://upload-images.jianshu.io/upload_images/5834071-75935e4d3196cc50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面这张图来源是简书里的另外一篇文章，简单的说客户端向服务端发送请求后，服务端会把自己的证书给客户端看，客户端信任了证书（公钥）之后，通过握手将协商密钥通过公钥加密发送，服务端用私钥解密并校验消息长度并返回给客户端，客户端校验消息长度一致完成后建立连接，双方开始用协商秘钥通过对称加密（比非对称加密速度快）。
Charles抓Https包的原理就是Charles做中间人，Charles信任远程网站的证书（比如百度https://www.baidu.com）,本地信任Charles的证书，然后本地发送的请求先和Charles建立连接，Charles再和百度建立连接进行请求，间接的获取了通信的数据。如果不经过任何设置，打开Charles，如下图
![初始情况.png](http://upload-images.jianshu.io/upload_images/5834071-203446fcfff97edf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时Charles没法获取https的数据包，这时设置一下Charles的SSL代理配置，加入www.baidu.com,https的端口号默认为443，加入后再次访问，

![修改CharlesProxy后.png](http://upload-images.jianshu.io/upload_images/5834071-214ebc94f86c2600.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![访问百度.png](http://upload-images.jianshu.io/upload_images/5834071-9cacf508ab47e7ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时发现可以访问，不过进入网址时会提示是否信任证书，点击继续访问，才会出现如下数据包，里面是进入百度的一些请求，浏览器上可以看到访问风险的图标，点开可以看到是Charles信任baidu的一个证书。
![证书.png](http://upload-images.jianshu.io/upload_images/5834071-a2faa8183b561283.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
浏览器中可以导入证书，信任Charles的根证书之后，正常情况下访问https相应的网站就不会有问题，同时还可以愉快的进行抓包。

## 2. 如何用手机配合Charles抓api
手机app可以很方便进行抓包，charles/help/SSLProxying下面有个install CA on remote device，点开如下
![charles](http://upload-images.jianshu.io/upload_images/5834071-6f355164f58db1c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里的Ip实际就是本机在局域网的ip和端口号，这些可以在cmd/ipconfig里面查看，打开手机，设置手机和电脑连接同一个网络，在网络下设置手动代理，proxy地址即为该提示的地址，保存网络
![网络设置.png](http://upload-images.jianshu.io/upload_images/5834071-9273352b0d1160d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时在手机上点开weibo，电脑上的charles即可看到请求，但是无法抓包，全是unknow，在SSL中添加api.weibo.cn:443，发现同样无法抓包。因为手机上要安装Charles根证书。本人用的是红米，在charles点击保存整数，保存为.pem格式（.cer格式证书无法读取），然后在手机上，wifi/高级设置/安装证书 通过文件查找到这个.pem的证书，起名CharlesProxy，即可。
再次点开微博，到电脑端查看，如下：
![微博抓包.png](http://upload-images.jianshu.io/upload_images/5834071-b5c214af6d3f9f84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就可以看到我们想要的api了，少量的爬取些数据都是ok的。

## 3.关于.cer/.pem有什么区别
在charles保存CA证书的时候，会发现两种格式，其实这两个文件都是证书，我的手机只能识别.pem的，当然也有些是可以识别.cer的，只是编码的方式不同。证书的作用也是在握手环节用于进行加密，建立连接。
![save.png](http://upload-images.jianshu.io/upload_images/5834071-8f53d4398cd7bfa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[参考文章](http://www.jianshu.com/p/870451cb4eb0)
[个人博客](https://j4fan.github.io/) 欢迎访问~
