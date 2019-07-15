---
title: 用python模拟知乎登录
date: 2017-08-18 10:33:00
tags: 
	- Python
	
---
>前言：最近看到公众号python之禅里面的历史文章，模拟登录知乎，又看到很多人在网上尝试写代码，自己也想试试，最新的验证码是选择倒立的汉字，本文采用手动输入验证码的形式完成验证，最后获取cookie，存入文件中，之后便可以无需登录直接访问了。
>所用python的包：requests，BeautifulSoup，json

首先之前使用python请求网页有层出不穷的包，但是用了requests之后才知道它有多方便，详情可以参考这篇文章，也是python之禅公众号的文章，这里帮忙推荐一波--->[链接](http://mp.weixin.qq.com/s/gO8E3lXZiL6_ql5rDuHwMQ)。
下面一步步开始讲解如何模拟登录。
## 1.观察请求参数（FireFox/Chrome）
在向网站发送请求的时候，服务器会对访问用户进行标识，以cookie的形式存放在浏览器，该浏览器下次进行访问时，则可以跳过登陆直接浏览页面，这是我们模拟知乎登陆的主要目的，获取cookie后，使用cookie访问主页面，则可以跳过登陆。在火狐中，先清掉之前保存的cookie，以模拟浏览器首次访问。
![清除火狐cookie](http://upload-images.jianshu.io/upload_images/5834071-9e6c660f8d81b98e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
清除cookie后，试着登陆知乎，发现有验证码，打开F12，一边发请求，一边观察网络中的变化。
这里故意输错密码，看看发送了什么请求。

![登陆请求头](http://upload-images.jianshu.io/upload_images/5834071-2679b4d38f31cd2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
![登陆请求参数](http://upload-images.jianshu.io/upload_images/5834071-500e37e330a78361.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

_xsrf的参数是用于防止xsrf攻击，如何防止xsrf攻击，可以参考[这篇文章](http://blog.csdn.net/newjueqi/article/details/7542409)，这是一种简单的恶意链接攻击，可以直接让你访问目标网站并且修改你的密码，所以一般在提交的表单中加个_xsrf参数，只有参数正确才可以进行提交。

## 2.代码模拟请求
可以看到知乎的主要请求参数，还有email,password,captcha,captcha_type。
自己试验过，captcha_type可以不用填写，首先还是找到验证码的链接如下：
https://www.zhihu.com/captcha.gif?r=1503020715890&type=login&lang=cn

![获取验证码的链接](http://upload-images.jianshu.io/upload_images/5834071-8a5e858b5322688d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
很明显，中间的数字是以毫秒为单位的时间戳，因此在代码中很容易模拟这个链接去发送请求获取验证码的图片。
```
def get_captcha():
    timestamp = str(time.time()*1000).split(".")[0]
    url = "https://www.zhihu.com/captcha.gif?r=%s&type=login&lang=cn" % timestamp
    print(url)
    req = session.get(url,headers=headers)
    with open("img1.png",'wb') as f:
        f.write(req.content)
```
我采用的方式是人工识别，所以把图片以二进制的形式写入文件，自己点开查看。
得到的内容是一行汉字：
![知乎验证码](http://upload-images.jianshu.io/upload_images/5834071-c31e380d68644a7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)
点击倒立的汉字，获取响应的坐标，放在请求参数中，即可模拟鼠标点击的过程，可以自己点击几个点，大概找到1234567个汉字的坐标，然后根据汉字的个数，匹配响应的坐标点，填入参数中即可。
```
def get_code():
    get_captcha()
    print("输入倒立汉字的位置，用逗号隔开")
    a = input("input:")
    indexs = a.split(",")
    data=[[16.4,26.9],[33.4,26.9],[60.4,21.9],[84.4,24.9],[108.4,24.9],[130.4,24.9],[156.4,24.9]]
    input_points = []
    for idx in indexs:
        input_points.append(data[int(idx)-1])
    dict = {
        "img_size": [200, 44],
        "input_points": input_points
    }
    return dict
```
有了xsrf参数和验证码，输入自己的账号密码，即可模拟登录
```
def login():
    url = "https://www.zhihu.com/login/email"
    data={
        "_xsrf":get_xsrf(),
        "email":'******@qq.com',
        "password":'asfgasdfasf',
        'captcha_type':get_code()
    }
    response = session.post(url, data=data,headers=headers)
    login_code = response.json()
    print(login_code['msg'])
    response = session.get("https://www.zhihu.com/settings/profile",headers=headers)
    #将cookies->dict->json存放在txt|之后再json->dict直接使用
    with open("cookies.txt",'w') as f:
        json.dump(session.cookies.get_dict(),f)
    with open("login.html","wb") as f:
        f.write(response.content)
```
第一次登录结束后，session.cookie会自动保持登录状态的cookie，下一次进行登录时，使用cookie即可，这里进行持久化，暂时写在txt文件中，下次使用时直接从文件获取。
这里有个小插曲就是session.cookie的类型是RequestCookie不是dict或者str，这里使用get_dict()方法转化成dict，并用json的形式存在txt文件中，取出来的时候，json可以直接提取成dict，放入session.cookie可以直接使用。
```
#将cookies->dict->json存放在txt|之后再json->dict直接使用
    with open("cookies.txt",'w') as f:
        json.dump(session.cookies.get_dict(),f)

#从txt文件中获取cookies,json->dict存入session.cookies
    with open("cookies.txt",'r') as f:
        cookies = json.load(f)
    session.cookies.update(cookies)
```
这里也可以温习下，Python中Json的操作->[链接](http://www.jianshu.com/p/26cb66297a6a)
获取cookie后，我尝试登录了(https://www.zhihu.com/settings/profile)这个地址，可以不需登录正常访问。
## 3.总结
写知乎登录，主要也是练练手，觉得挺有意思，后面可以尝试用phantomjs+selenium模拟下登录爬取，有了登录cookie之后，也可以爬一下知乎的内容，应该有很多有趣的内容，后续会继续更新的，谢谢关注~
github上我也上传了代码，欢迎大家点评，学习。
[源码地址](https://github.com/Fan4J/Spiders-of-frequently-used-website/tree/master/zhihuLogin)
[重塑自我](https://fan4j.github.io/)
