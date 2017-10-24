---
title: Enum类使用详解
date: 2017-10-24 17:26:09
tags: 
	- Java
comments: true
---

>今天自己研究RestApi，其中对请求的响应结果封装成Result类，Result类里面又是三个参数,int类型的code,String类型的message,和object类型的data,其中code是枚举类封装的，特此学习一下，枚举类还是很有用的。

## 1.项目代码中对响应code的封装
先上代码
```
public enum ResultCode {
    SUCCESS(200),//成功
    FAIL(400),//失败
    UNAUTHORIZED(401),//未认证（签名错误）
    NOT_FOUND(404),//接口不存在
    INTERNAL_SERVER_ERROR(500);//服务器内部错误

    int code;

    ResultCode(int code) {
        this.code = code;
    }
}
```
枚举类的应用场景：通常用来列举一个类型的有限实例集合，例如颜色，日期，代码等等。
java.object包下的Enum可以用idea查看下源码，下面有很多子类，如下

![Enum子类.png](http://upload-images.jianshu.io/upload_images/5834071-08cf731d9b97b188.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
随便打开一个，都是enum枚举类，代表着各种适合枚举的实例

![MemoryType.png](http://upload-images.jianshu.io/upload_images/5834071-cba8a47b0d36207b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对我们项目中的ResultCode类进行编译，然后反编译，如下：
```
javac ResultCode.java #编译得到.class文件
javap ResultCode.class #反编译
```
结果如下
```
Compiled from "ResultCode.java"
public final class ResultCode extends java.lang.Enum<ResultCode> {
  public static final ResultCode SUCCESS;
  public static final ResultCode FAIL;
  public static final ResultCode UNAUTHORIZED;
  public static final ResultCode NOT_FOUND;
  public static final ResultCode INTERNAL_SERVER_ERROR;
  int code;
  public static ResultCode[] values();
  public static ResultCode valueOf(java.lang.String);
  static {};
}
```
可以看到编译后的enum类其实是一个final的class，而且默认集成了java.lang.Enum，因此任何enum类是不能被继承，也不能再继承其他类的。通过这个可以看出，枚举类中包含N个该枚举类的静态的final实例。

## 2 简单的enum类
```
public enum Week {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}

```
在一个TestClass里

![Enum方法.png](http://upload-images.jianshu.io/upload_images/5834071-ec4dcbf913e2ff2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
public static void main(String[] args) {
        Week week = Week.FRI;
        System.out.println(week.ordinal());
        System.out.println(week.compareTo(Week.THU));
        System.out.println(week.name());
        System.out.println(Week.valueOf("FRI"));
        System.out.println(Week.values());
    }
```
输出如下
```
5
1
FRI
FRI
[Lcom.j4fan.JiCheng.Week;@2b193f2d
```
其中.ordinal()方法是输出枚举类该对象的下标，compareTo()方法也很简单，直接使用下标进行比较
源码如下，都比较简单。name()和toString()方法都是返回name，因此结果也是一样的，都是该对象的name，还有个静态方法values()，返回该enum下所有的静态实例，是个隐式的方法。
![compareTo方法.png](http://upload-images.jianshu.io/upload_images/5834071-5bce54d91e0929ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![values()](http://upload-images.jianshu.io/upload_images/5834071-a929957c51713e11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.带有变量的enum类
先上代码
```
public enum Weekdays {
    SUN(0), MON(1), TUE(2), WED(3), THU(4), FRI(5), SAT(6);
    int value;

    Weekdays(int value) {
        this.value = value;
    }

    public static Weekdays getNextDay(Weekdays day) {
        if (day.value == 6) {
            return Weekdays.SUN;
        } else {
            return getWeekDaysByValue(day.value + 1);
        }
    }

    public static Weekdays getWeekDaysByValue(int value) {
        for (Weekdays c : Weekdays.values()) {
            if (c.value == value) {
                return c;
            }
        }
        return null;
    }
}
```
用法还是很简单的，其次变量可以更加丰富，比如code+message，只需进行封装即可；
```
public enum WeekNote {
    MON(0,"work"),
    TUE(1,"play"),
    WED(2,"talk"),
    THU(3,"learn"),
    FRI(4,"read"),
    SAT(5,"laugh"),
    SUN(6,"greet");

    int code;
    String message;

    WeekNote(int code,String message){
        this.code = code;
        this.message = message;
    }
}
```

## 4.向enum类中添加方法
其实enum可以对属性加上get/set方法，这里我产生了疑问，既然每个enum实例是final类型的，为什么还可以set呢，这里我做了个实验
```
public class TestClass {
    private static final User user = new User("fan","male");

    public static void setName(String s) {
        user.setName(s);
    }

    public User getUser(){
        return user;
    }

    public static void main(String[] args) {
        TestClass t = new TestClass();
        t.setName("jiang");
        System.out.println(t.getUser().getName());
    }
}
```
这里在方法内部new一个final对象，运行结果证明确实属性被修改了，查找资料，发现我对final的理解有误，对于变量来说，是指不可修改，对于对象来说，是对象的引用不可修改，以后一定要注意。

## 5.enum的其他用法
enum类虽然是继承了Enum方法，但是还是可以实现其他接口的，因此可以在enum类中加入其他的接口，添加对应的实现。偷懒贴段别人的代码
```
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
//接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}  
```
## 总结
enum枚举类在项目中应用还是挺多的，其实它可以用一个常量类代替，常量类里面都是private static final类型的静态变量，但是容易出现有的朋友调用没有的变量，或者传入一个非主流的code，导致特殊的异常，enum还是有用武之地的。弄完这些，继续去研究restapi了，撤~
[个人博客](https://fan4j.github.io/)欢迎访问