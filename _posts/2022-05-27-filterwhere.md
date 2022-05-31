---
layout:     post
title:      spark - select where or filtering? withColumn 
subtitle:   用于过滤/用于添加新字段、字段重命名、修改字段类型  
date:       2022-05-27
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  

## [select where or filtering?](https://stackoverflow.com/questions/38867472/spark-select-where-or-filtering)  
<font color=red>answer 1：</font>        
As Yaron mentioned, there isnot any difference between **where** and **filter**.  
**filter** is an overload method that takes a column or string argument. The performance is the same, <font color=red>regardless of the syntax you use.</font>  
we can use **explain()** to see that all the difference filtering syntaxes generate <font color=red>the same Physical Plan</font>.  

The syntax doesnot change how filters are executed under the hood,but the file format/database that a query is executed on does. Spark will execute the same query differently on Postgres(predicte pushdown filtering is supported), Parquet(column pruning), and CSV files.    
<font color=red>answer 2：</font>  
According to **spark documentation** "where() is an alias for filter()".  
**filter(condition)** Filters rows using the given condition. **where()** is an alias for **filter()**.  

## [Explain where filter using dataframe in Spark](https://www.projectpro.io/recipes/explain-where-filter-dataframe-spark)    
The Spark where() function is defined to filter rows from the dataFrame or the dataset based on the given one or multiple conditions or SQL expression. The where() operator can be used instead of the filter when the user has the SQL background. Both the where() and filter() operate precisely the same. Using the where() function,<font color=red> single column or multiple columns can be output in spark using the basicsyntax with NULL conditions.</font> Using the where() function,<font color=red> the array and nested structure can be output in spark using the basic syntax with false conditions.</font>  

<font color=green>If you want to provide a filter on multiple columns, you can do it using AND(&&) or OR(||). You can use the filter() function multiple times to achieve the same.</font>  
## 语法  
**filter**  
```
df.filter($"key1">"aaa").show()
df.filter($"key1"==="aaa").show()
df.filter("key1='aaa'").show()
df.filter("key2=1").show()
df.filter($"key2"===3).show()
df.filter($"key2"===$"key3"-1).show()
```  

其中, ===是在Column类中定义的函数，对应的不等于是=!=。
$"列名"这个是语法糖，返回Column对象.  

**where**  
```
df.where("key1='vvvv'").show()
df.where($"key2"=!=2).show()
df.where($"key3">col("key2")).show()
df.where($"key3">col("key2") + 1).show()
```  

## spark 的一些 ETL 操作  
```scala
val sys_company = admin.repartition(10)
      .join(orgfunction,admin("forgfunctionid")===orgfunction("fid"),"left")
      .join(reserveitemfirst,admin("freserveitemfirst")===reserveitemfirst("fid"),"left")
      .select(col("admin.fnumber"),col("admin.forgfunctionid"),col("admin.flongnumber"),col("admin.fsortcode"),
        col("admin.freserveitemfirst"),
        col("orgfunction.fname_l2").as("systemcategory"),
        col("orgfunction.fnumber").as("systemnumber"),
        col("reserveitemfirst.fname_l2").as("companycategory"),
        col("reserveitemfirst.fnumber").as("companynumber"))
      .withColumn("systemcategory",when(col("systemcategory")===null,"其他").otherwise(col("systemcategory")))
      .withColumn("companycategory",when(col("companycategory")===null,"其他").otherwise(col("companycategory")))
```  

```
val emp = spark.table("gree_screendisplay_hr.hr_emp_info").filter(dateFilterContition).withColumn("isleave",
      when(col("entrydate") <= col("date") and col("leavedate").isNull,false)
        when(col("entrydate")<=col("date") and(col("leavedate")>col("date")),false)
        when(col("entrydate")>=col("date") or(col("leavedate")<=col("date")),true)
        otherwise(col("isleave"))

```  
## [Spark DataFrame withColumn](https://sparkbyexamples.com/spark/spark-dataframe-withcolumn/)  
Spark **withColumn()** is a DataFrame function that is used to add a new column to DataFrame, change the value of an existing column, convert the dataType of a column, derive a new column from an existing column.  
* Spark withColumn() Syntax and Usage  
Spark withColumn is a transformation function of DataFrame that is used to manipulate the column values of all rows or selected rows on DataFrame.  
**withColumn** function returns a new spark dataframe after performing operations.  
* Add a new column to DataFrame  
```
// lit() function is used to add a constant value to a DataFrame column.
import org.apache.spark.sql.functions.lit
df.withColumn("country", lit("USA"))
df.withColumn("country", lit("USA")).withColumn("anotherColumn", lit("anotherColumn"))
```  
> The above approach is fine if you are manipulating few columns, but you wanted to add or update multiple columns, do not use the chaining withColumn() as it leads to performance issues, use select() to update multiple columns instead.  
* Change value of an Existing Column  
```
import org.apache.spark.sql.functions.col
df.withColumn("salary", col("salary")*100)
```  
* Derive new Column from an Existing column  
```
df.withColumn("copiedColumn", col("salary")*-1)
```  
* Change column Data Type  
```
df.withColumn("salary", col("salary").cast("Integer"))
```  
* Add, Replace,or Update multiple columns  
```
df2.createOrReplaceTempView("PERSON")
spark.sql("SELECT salary*100 as salary, salary*-1 as CopiedColumn, 'USA' as country FROM PERSON").show()
```  
> When you wanted to add, replace or update multiple columns in Spark DataFrame, it is not suggestible to chain withColumn() function as it leads into performance issue and recommend to use select() after creating a temporary view on DataFrame.
## [Spark withColumn陷阱](https://blog.csdn.net/lsshlsw/article/details/105802839)  
withColumn/withColumnRenamed 是spark 中常见的 API，可以用于添加新字段、字段重命名、修改字段类型，但当列的数量增加时，会出现严重的性能下降现象。  

**df.explain(true)**：查看计划（物理计划、逻辑计划）  

```scala
import org.apache.spark.sql.catalyst.rules.RuleExecutor
println(RuleExecutor.dumpTimeSpent())
```  
以上代码用于查看 **catalyst analysis** 的统计信息。  

综上，多次执行 withColumn/withColumnRenamed 时，大部分时间会花费在 catalyse analyse 的反复调用上，且随着迭代次数的增加，逻辑计划的project会增加，耗时会呈指数上升，具体的耗时还会随原表字段数进行一些变化。  










  
  



 
