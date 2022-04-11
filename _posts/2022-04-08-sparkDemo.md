---
layout:     post
title:      spark的二进制原始数据解析方法  
subtitle:   spark实现大量数据的快速解析并写入hive表  
date:       2022-04-08
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  
原始数据为二进制数据，存储在本地MySQL5.7中，欲拿来进行数据分析需要提前进行解析。鉴于数据量较大，因此使用spark的解析方法。  
申请资源：50cores   300G   
# spark连接MySQL解析测试  
首先，连接单张表进行测试：  
```
spark.read.format("jdbc")
.option("url","jdbc:mysql://localhost:3306/db?useSSL=false")
.option("driver","com.mysql.jdbc.Driver")
.option("dbtable","xxx")
.option("user","root")
.option("password","xxx")
.load()
.show()
```  
这样，我们就获得了MySQL表的所有数据。接下来，对相关字段transfer_data进行解析，同时注意到show的结果为dataframe，因此使用sparksql DSL 的解析方法。    
对解析部分构建用户自定义函数UDF：  

val dispose_spark_udf = udf(new SparkUdf().dispose_spark_map(_:Array[Byte]):mutable.Map[String,String])  
结果，当数据量较大时，运行速度特别慢。  
# 单表解析优化  
显然，我们需要对代码进行优化，即对jdbc读取并发度优化，[这是因为单线程任务过重导致，需要提高读取的并发度。](https://yerias.github.io/2020/11/05/spark/36/#%E6%BA%90%E7%A0%81)  
查看：  
```
DF.rdd.partitions.size
```
优化前的操作并发度为1，即所有的数据都会在一个partition中进行操作，意味着无论你给多少资源，只有一个task会执行任务，执行效率可想而知，并且在稍微大点的表中进行操作分分钟就会OOM。  
```
.option("partitionColumn","id")
.option("lowerBound",1)
.option("upperBound",100000)
.option("numPartitions",20)
```  
**缺点：**只适合读取数字型的主键或者分区写死。  
**注：**单线程任务过重，数据库数据较大时任务会hang住；若分区设置较高时（大量partition同时读取）也可能造成将数据源数据库弄挂。  
# MySQL分页读取方法优化   
```
"transfer_time in (select transfer_time from ${tableName} " +
          s"where transfer_time >= DATE_ADD(DATE_ADD('${date_transfromation}', INTERVAL -1 day), INTERVAL ${i} hour) " +
          s"and " +
          s"transfer_time < DATE_ADD(DATE_ADD('${date_transfromation}', INTERVAL -1 day), INTERVAL ${i+1} hour)) " +
          s"and " +
          s"length(transfer_data)=107"

sparksession.read.jdbc("url",tableName,partitionArray,prop)
```  
将分区从24个提升值48个，发现sparksql DSL 的解析方法速度还是慢，3240条数据，约1h。  
# dataframe的map优化操作    
```
val result = dataDF.map(x => {
	"""
	解析方法
    """
})
```  
发现解析速度提升了近240倍，仅用了15s解析保存完成。  
# spark RDD 方法  
同样，可以使用spark rdd的方法进行解析，但该方法使用 df.collect 收集了所有分区数据至driver端。  
**collect算子：将RDD类型的数据转化为数组，为action算子。一次collect会导致一次shuffle，因为需要将所有数据拉倒driver端**  
此外，当数据量较大时，因为driver端内存的问题，可能造成OOM等问题。  
舍弃。。。   
# 分析原因  
通过上述优化方法，注意到sparksql DSL（UDF用户自定义函数） 解析速度较dataframe map操作慢，这是为什么呢？尝试搜索答案。  
[Spark functions vs UDF performance?](https://stackoverflow.com/questions/38296609/spark-functions-vs-udf-performance)中提到：  

When executing Spark-SQL native functions, the data will stays in <font color=red>tungsten backend</font>. However, in Spark UDF scenario, the data will be moved out from tungsten into JVM (Scala scenario) or JVM and Python Process (Python) to do the actual process, and then move back into tungsten.   

Unlike UDFs, Spark SQL functions operate directly on JVM and typically are well integrated with both Catalyst and Tungsten. It means these can be optimized in the execution plan and most of the time can benefit from codgen and other Tungsten optimizations. Moreover these can operate on data in its "native" representation.  

(pyspark)The main reasons are already enumerated above and can be reduced to a simple fact that Spark DataFrame is natively a JVM structure and standard access methods are implemented by simple calls to Java API. UDF from the other hand are implemented in Python and require moving data back and forth.  

综上所述，首选使用sparksql以及spark sql nativate functions。  

**spark 内存管理之Tungsten**    
Tungsten 号称spark有史以来最大改动，其致力于提升spark程序对内存和CPU的利用率，使性能达到硬件的极限，主要工作包括以下三个方面：  
1. Memory management and Binary Processing: off-heap管理内存，降低对象的开销和消除JVM GC带来的延迟；  
2. Cache-aware computation: 优化存储，提升CPU L1/L2/L3 缓存命中率；  
3. Code generationL 优化 spark SQL的代码生成部分，提升CPU利用率。  
 

  
  
 






