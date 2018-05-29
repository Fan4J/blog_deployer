---
title: ubuntu之初体验
date: 2018-05-29 14:13:09
tags: Linux
comments: true
---
>进来为了研究hadoop,心血来潮的把自己家里pc装了ubuntu.刚开始不太会用,这里记录下一些经验

## 1 软件安装顺序
1.java jdk 环境变量
2.python
3.idea pycharm
4.搜狗拼音
5.sublimetext
6.shutter 暂时不太会用

## 2 安装软件
通常的方式有yum apt-get 例如
```
yum install python3.4
sudo apt-get python
```
如果是yum库和ubuntu软件库没有添加的软件可以下载源码安装
```
tar -xf archive.tar
```
解压后放在自己熟悉的目录我一般是
```
sudo mv ~/Downloads/archive /usr/local/share
```
一般我会做两步 1.通过加软链接,实现终端打开软件 2.创建桌面图标,桌面打开 eg
```
ln -s /usr/local/share/sublime_text_3 /usr/bin/sublime
```
完成后在终端的任何地方输入sublime即可打开
```
cd  /usr/share/applications
vi  sublime.desktop
```
进入编辑 替换Name为软件的名字 Icon为图标的图片地址 Exec 是执行脚本的位置
```
[Desktop Entry]

Type=Application

Name=SublimeText

Icon=/usr/local/share/sublime_text_3/Icon/256x256/sublime-text.png

Exec=/usr/local/share/sublime_text_3/sublime_text

Terminal=false

Categories=Network;InstantMessaging
```
完成以后可以在应用程序中看到,点击收藏即可显示在左边侧边栏了

![屏幕截图.png](https://upload-images.jianshu.io/upload_images/5834071-9f2cb874755d2871.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3 切换繁体简体
Ctrl+Shift+F
