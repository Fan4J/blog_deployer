---
title: nginx搭建图片服务器
date: 2019-07-16 15:52:36
tags: 
	- 系统架构

---
>用有道云笔记写笔记需要贴图片的时候，发现需要会员才能直接贴图片，因此想到做一个简易版的图片服务器。

1.下载安装(这个教程是mac上的，实际部署是在阿里云的一台ECS服务器上)
---
我的电脑是mac,下面的教程是在mac上安装Nginx<br>

如果是Linux的机器，可以参考这个教程[菜鸟教程](https://www.runoob.com/linux/nginx-install-setup.html)<br>

打开官网，点击下载链接<br>

[官网下载链接](http://nginx.org/en/download.html)

下载得到一个文件夹
```
~/Document/nginx-1.17.1.tar.gz
```
解压
```
tar zxvf nginx-1.17.1.tar.gz
```
进入目录并install
```
./configure
make 
make install
```
没有报错即安装成功,查看版本
```
/usr/local/nginx/sbin/nginx -v 
```
得到结果: 
```
nginx version: nginx/1.17.1
```

2.nginx目录
---
```
cd /usr/local/nginx/
ls
```
![nginx目录](http://47.100.121.52/images/nginx-content.png)

- conf配置文件目录
- logs日志文件目录
- html静态文件目录
- sbin执行文件目录

3.配置
---
打开conf文件下的nginx.conf文件，进行配置。
```
//user一般为root或者不填写这行
#user  nobody;

//worker的进程数一般和CPU核心数保持一致
worker_processes  1;

//各种log的地址
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

//nginx运行时进程id存放位置
#pid        logs/nginx.pid;

//工作最大连接数
events {
    worker_connections  1024;
}
```

接下来是serverblock,如果nginx需要监听多个端口做代理，则可以写多个serverblock
```
    http {
    server {
    }
}
```

默认nginx是listen80端口的，我们做如下配置
```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```
location中的的前缀匹配具有优先匹配最长路径<br>
访问路径为localhost/images/a.png -> /date/images/a.png<br>
访问路径为localhost/a.html -> /data/a.html

第一次启动需要
```
/usr/local/nginx/sbin/nginx 
```
修改配置后更新配置
```
/usr/local/nginx/sbin/nginx -s reload
```

4.测试图片服务器
---
我们点开浏览器,访问http://ip/,可以看到nginx的欢迎界面如下:
![nginx欢迎界面](http://47.100.121.52/images/welcome.png)

后面继续加上我们刚刚上传的文件名，即可看到对应的图片，访问http://ip/avatar.jpg,如图所示

![头像](http://47.100.121.52/images/avatar.jpg)

5.结尾
---
至此，我的简易版nginx图片服务器完成了。

后续还有待办的事项需要实现:
- 完成域名备案，将地址换成域名请求而不是ip(备案目前还需要十几天)
- 做一个简单的页面和Java后端程序做上传下载，搜索等功能(目前是用idea的aliyun-cloud-toolkit直接上传的)
