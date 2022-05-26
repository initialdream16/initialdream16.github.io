---
layout:     post
title:      spark 中的 dataframe 和 dataset 
subtitle:   sparkSQL 中的两种数据类型  
date:       2022-05-26
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  
## [Spark SQL概述](https://blog.csdn.net/howard2005/article/details/124341503)     
是一种用于结构化数据处理的spark组件。所谓结构化数据，是指具有schema信息的数据。  
## Spark SQL主要特点  
-1. 将SQL查询与spark应用程序无缝组合  
-2. Spark SQL以相同方式连接多种数据源  
-3. 在现有数据仓库上运行SQL或HiveQL查询  
## DataFrame  
是 Spark SQL 提供的一个编程抽象，也是一个分布式的数据集合。dataframe 在RDD的基础上添加了数据描述信息（Schema，即元信息），因此看起来更像是一张数据库表。  
## DataSet  
是一个分布式数据集。相对于RDD，DataSet 提供了强类型支持，在RDD的每行数据加了类型约束。  

<table><tr><td bgcolor=yellow>**使用DataFrame API和Dataset API 均可以经过 Spark SQL优化器的优化，从而提高程序执行效率。**</td></tr></table>  

## sparkSession.read.textFile 和 read.json  

```scala
val df = sparksession.read.json("xxx\\people.json")
```  
*df 数据类型：DataFrame*   


```scala
val df1 = sparksession.read.textFile("xxx\\people.txt")
```
*df1 数据类型：DataSet*   

## [row dataframe dataset 间的相互转化](https://blog.csdn.net/qq_42456324/article/details/124587709)  

```
Row --> DataFrame        .toDF
Row --> DataSet          .toDS
DataFrame --> DataSet    .as[Subject]
DataSet --> DataFrame    .toDF
DataFrame --> Row        .rdd
DataSet --> Row          .rdd
```  












  
  



 