---
layout:     post
title:      Java 设计模式之单例模式
subtitle:   
date:       2022-01-06
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - Java
---  

# Java 设计模式之单例模式  
## 概述  
Java Singleton 模式的定义就是确保某一个类 class 只有一个实例，并且提供一个全局访问点。属于设计模式三大类中的**创建型模式**。  
典型特点：  
* 只有一个实例   
* 自我实例化  
* 提供全局访问点  

## 优缺点  
* 优点：由于单例模式只生成了一个实例，所以能够节约系统资源，减少性能开销，提高系统效率，同时能够严格控制客户对它的访问  
* 缺点：也正是因为系统中只有一个实例，这样就导致了单例类的职责过重，违背了“单一职责原则”，同时也没有抽象类，这样扩展起来有一定的困难。  

## [五种实现方式](https://www.cnblogs.com/ngy0217/p/9006716.html)  
* 饿汉式：线程安全，调用效率高。但是不能延时加载。 加载类时的速度比较慢，但是获取对象的速度比较快。 
```
public class SingletonDemo1{
  // 线程安全
  // 类初始化时，立即加载该对象
  private static SingletonDemo1 instance = new SingletonDemo1();
  private SingletonDemo1(){}
  
  // 方法没有加同步块，所以它的效率高
  public static SingletonDemo1 getInstance(){
    return instance;
  }
}
```  

* 懒汉式：线程不安全。加载类比较快，但是对象的获取速度相对较慢。 
```
public class SingletonDemo2{
  private static SingletonDemo2 instance = null;
  private SingletonDemo2(){}
  
  // 运行时加载对象
  public static SingletonDemo2 getInstance(){
    if(instance == null){
      instance = new SingletonDemo2();
    }
    return instance;
  }
}
```  
* Double CheckLock  
* 静态内部类实现模式(线程安全，调用效率高，可以加载延时)  
* 枚举类(线程安全，调用效率高，不能延时加载，可以天然的放置反射和反序列化调用)  
  
## 常见应用场景  
* 网站计数器  
* 项目中用于读取配置文件的类  
* 数据连接池  
* spring中，每个 bean 默认都是单例的  
* Servlet 中 application  
* windows 中任务管理器，回收站
* ……  

## 参考资料  
1. [java单例模式几种实现方式](https://www.cnblogs.com/ngy0217/p/9006716.html)  
2. [Java设计模式（一）之单例模式](https://www.jianshu.com/p/3f5eb3e0b050)  
3. [Java中用单例模式有什么好处?](https://www.cnblogs.com/tangxiao1996/p/7899393.html)
