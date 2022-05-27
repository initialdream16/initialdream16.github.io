---
layout:     post
title:      scala 异常（Exception）/throws关键字 
subtitle:    
date:       2022-05-27
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  

## java 中异常处理方式  
```java
try{
}case(异常类型1){
	// 异常处理
}case(异常类型2){
	// 异常处理
}finally{

}
```  
捕获异常：  
try{..}catch(..){..}finally{...}  
抛出异常：  
-1. 在方法体中通过 throw 关键字抛出  
-2. 在方法名后面通过 throws 关键字声明异常  
## scala 异常（Exception）处理方式    
捕获异常：  
-1. try{...}catch{case e:Exception => ...}finally{...}  
-2. Try(代码块).getOrElse（默认值）  
抛出异常：  
在方法体中通过throw关键字抛出  

```scala
val a =1
val b =0
try{
	a/b
}catch{
	case e: ArithmetricException => println("被除数不能为零")
}
```  
**案例：**统计员工平均年龄，遇到报错信息：age 数字转换异常（为空）

**方案1：** 使用 if 条件
```scala
val sum = list.map(s => {
	val infoArray = s.split("\t")
	val age = infoArray(2)
	if(age != "") age.toInt else 0
}).reduce((x,y) => x+y)
```  

**方案2：**使用 try...catch 方式  
```scala 
val sum = list.map(s =>{
	val infoArray=s.split("\t")
	val age = infoArray(2)
	try{age.toInt}catch{case NumberFormatException => 0} 
}).reduce((x,y) => x+y)
```  

**方案3：**使用scala.util包  
```scala
import scala.util.Try
val sum = list.map(s =>{
	val infoArray=s.split("\t")
	val age = infoArray(2)
	Try(age.toInt).getOrElse(0)
}).reduce((x,y) => x+y)
```  
## [scala  throw throws 关键字](https://blog.csdn.net/love284969214/article/details/82731776)    
scala 提供了 throw 关键字来抛出异常。throw 关键字主要用于抛出自定义异常。同时也提供了 throws 关键字来声明异常。可以使用方法定义声明异常。它向调用者函数提供了次方法可能引发此异常的信息。它有助于调用函数处理并将该代码包含在try...catch块中，以避免程序异常终止。在scala中，可以使用 throws 注释来声明异常。  

示例1：  
```
class ExceptionExample2{
	def validate(age:Int)={
		if(age < 18)
			throw new ArithmetricException("you are not eligible")
		else println("you are eligible")
	}
}

object MainObject{
	def main(args: Array[String]){
		var e = new ExceptionExample2()
		e.validate(10)
	}
}
```  

示例2：  
```
class ExceptionExample2{
	@throws(classOf[NumberFormatException])
	def validate()={
		"abc".toInt
	}
}

object MainObject{
	def main(args: Array[String]){
		var e = new ExceptionExample4()
		try{
			e.validate()
		}catch{
			case ex: NumberFormatException => println("exception handeled here")
		}
		println("Rest of the code executing...")	
	}
}
```
















  
  



 