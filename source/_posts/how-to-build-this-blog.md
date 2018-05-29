---
title: 用GithubPages+Hexo搭建漂亮的博客
date: 2017-07-25 09:04:14
tags: 
	- 技术
	- 博客

---
>一直以来都想自己搭建博客，作为个人生活和技术的记录，终于在前几天开始动手，一天多时间搭建了自己的个人博客，搭建方式：github+hexo（注：没有绑定个人域名，直接通过github提供的个性链接访问），这种方式可能会在速度上由于github本身的一些原因受到影响，存储空间上其实github没有什么限制，大可放心使用。
本文主要记录搭建的过程以及遇到的一些坑，仅供大家参考。[浏览博客](https://fan4j.github.io/)

## 1.准备Github,git,nodejs
第一步，注册github账号，安装git,nodejs
github地址:https://github.com/
git下载地址:https://git-scm.com/downloads
nodejs下载地址:http://nodejs.cn/ 
nodejs的学习可以参考官方文档:http://nodejs.cn/api/
git如果不会建议先看廖雪峰的git教程:[点击链接](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
一切就绪后可以在存放代码的目录下打开git bash,确认是否安装成功
```
git
node -v
npm -v  //最新版的nodejs已经集成了npm包
```
## 2.初始化hexo
依次进行安装hexo,查看是否安装成功，初始化博客，部署，如下
```
hexo init hexo  //初始化目录
cd hexo  //进入目录
hexo generate  //生成html    
hexo server  //运行本地服务
```
在本地部署完成之后，可以在
http://localhost:4000/
查看部署效果，目录结构如下：
```
├── .deploy       //需要部署的文件
├── node_modules  //Hexo插件
├── public        //生成的静态网页文件
├── scaffolds     //模板
├── source        //博客正文和其他源文件, 404 favicon CNAME 等都应该放在这里
|   ├── _drafts   //草稿
|   └── _posts    //文章
├── themes        //主题
├── _config.yml   //全局配置文件
└── package.json
```
## 3.修改_config.yml进行格式修改
主要配置如下，可以用于参考，需要注意的是yml格式非常严格，冒号后面需要加空格，后面jsContent是为了yilia标签库特效准备的
```
# Site
title: 重塑自我
subtitle: 当你低头的瞬间，才发现脚下的路
description: 当你低头的瞬间，才发现脚下的路
author: Fan4j
language: zh-Hans
timezone: Asia/Shanghai

deploy:
  type: git
  repo: git@github.com:Fan4J/fan4j.github.io.git
  branch: master
  
jsonContent:
    meta: false
    pages: false
    posts:
      title: true
      date: true
      path: true
      text: false
      raw: false
      content: false
      slug: false
      updated: false
      comments: false
      link: false
      permalink: false
      excerpt: false
      categories: false
      tags: true
```
## 4.修改主题
进入主题目录下，从github上找自己喜欢的好看的hexo主题，并下载。
hexo主题地址:https://hexo.io/themes/
点击到相应的github的主页，找到项目的地址，克隆到本地，比如我的博客使用的很受欢迎的yilia主题，如下
```
cd hexo
cd themes //cd到themes目录下
git clone https://github.com/litten/hexo-theme-yilia.git yilia //下载到yilia文件夹
```
主题的目录结构如下
```
├── languages       //主题的语言包
├── layout  //主题样式
├── source       // 资源文件目录，例如图片
├── source-src     //
├── _config.yml   //配置文件
└── package.json
```
主要修改在于_config.yml配置
主要配置节选如下
```
# Header

menu:
  主页: /
  随笔: /tags/随笔/
  技术: /tags/技术/
  
# SubNav
subnav:
  github: "https://github.com/Fan4J"
  weibo: "http://weibo.com/2179165162/profile?topnav=1&wvr=6"
  #rss: "#"
  zhihu: "https://www.zhihu.com/people/storm-spirit/activities"
  #qq: "#"
  #weixin: "#"
  jianshu: "http://www.jianshu.com/u/c4f8f0c4a19f"
  #douban: "#"
  #segmentfault: "#"
  #bilibili: "#"
  #acfun: "#"
  #mail: "fspirit@yeah.net"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

# 打赏
# 打赏type设定：0-关闭打赏； 1-文章对应的md文件里有reward:true属性，才有打赏； 2-所有文章均有打赏
reward_type: 2
# 打赏wording
reward_wording: '多谢支持,一起努力~！'
# 支付宝二维码图片地址，跟你设置头像的方式一样。比如：/assets/img/alipay.jpg
alipay: /img/payali.jpg
# 微信二维码图片地址
weixin: /img/paywechat.jpg


smart_menu:
  innerArchive: '所有文章'
  friends: '友链'
  aboutme: '关于我'

friends:
  w3school: http://www.w3school.com.cn/
  hexo: https://hexo.io/
  hexo-theme-yilia: https://github.com/litten/hexo-theme-yilia/
  

aboutme: 为了美好生活而拼命努力的普通人
```
添加头像既可以用本地的资源路径，记得是theme文件夹下的路径，例如我的图片放在theme/img/目录下
```
avatar: /img/head.jpg
```
打赏的图片同理
同时对于标签的js-content，需要安装一个包才可不出错，记着是在hexo目录下操作
```
cd hexo
npm i hexo-generator-json-content --save
```
## 5.绑定Github并部署
GithubPages是指特定命名方式的respository: namespace.github.io,比如我的就是fan4j.github.io
这样命名的仓库会被自动解析为githubpage,只需在hexo的全局配置文件中正确配置，即可简单部署
```
npm install hexo-deployer-git --save
npm deploy 
```
## 6.发布新文章
以上内容弄好以后，发布新文章就很轻松了
```
cd hexo
hexo new "posttitle"  //会在hexo/source/_posts目录下出现新的文章
```
打开文章，添加内容，修改配置，通常打开markdown文件可看到文件头，可以添加相应的文章标签，
```
title: 用GithubPages+Hexo搭建漂亮的博客
date: 2017-07-25 09:04:14
tags: 
	- 技术
	- 博客
```
tags可以继续添加，修改完成后即可开始部署到github服务器
```
hexo generate 
hexo deploy
```
部署完成，以后用这种方法部署即可
hexo现在也支持简写，构建过程更加简易
```
hexo g == hexo generate
hexo d == hexo deploy
hexo s == hexo server
hexo n == hexo new
```
## 7.代码保存，其他配置
这时打开github可以看到仓库中已经是前端页面的代码，说明部署完毕，访问地址即可-> https://fan4j.github.io/
同时可以将部署内容保存到新的respository中，可以在多个电脑上进行新文章的发布部署。
## 8.结束语
部署结束，中间可能遇到些坑，浪费了些时间，但是都还是可以通过网上搜索获取答案，总体还是觉得很值的，最后套用某人的话，希望不是三分钟热度，一定要把这个博客坚持下去~