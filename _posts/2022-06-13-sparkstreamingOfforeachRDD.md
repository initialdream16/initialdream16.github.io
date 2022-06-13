---
layout:     post
title:      sparkstreaming 之foreachRDD
subtitle:   
date:       2022-06-13
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  
[sparkStreaming 之 foreachRDD](https://blog.csdn.net/legotime/article/details/51836039)  
DStream 中的 foreach 是一个非常强大的函数，它允许你将数据发给外部系统。  

```
dstream.foreachRDD{rdd => 
	val connection = createNewConnection()   // executed at the driver
	rdd.foreach{
		connection.send(record)    // executed at the worker
	}
}
```  

> 这是不正确的，因为这需要先序列化连接对象，然后将它从driver发送到worker中。这样的连接对象在机器之间不能传送。它可能表现为序列化错误或者初始化错误等。正确的解决方法是在worker中创建连接对象。  

```
dstream.foreachRDD{rdd => 
	rdd.foreach{record =>
		val connection = createNewConnection()
		connection.send(record)
		connection.close()
	}
}
```  

> 这会造成另外一个常见的错误，为每一个记录创建了一个连接对象。通常，创建一个连接对象有资源和时间的开支。因为，为每个记录创建和销毁连接对象会导致非常高的开支，明显的减少系统的整体吞吐量。  

```
dstream.foreachRDD{rdd =>
	rdd.foreachPartition{partitionOfRecords => 
		val connection = createNewConnection()
		partitionOfRecords.foreach(record => connection.send(record))
		connection.close()
	}
}
```  

> 可以通过在多个RDD或者批数据间重用连接对象做更进一步的优化。开发者可以保有一个静态的连接对象池，重复使用池中的对象将多批次的RDD推送到外部系统，以进一步节省开支。  

```
dstream.foreachRDD{rdd =>
	rdd.foreachPartition{partitionOfRecords =>
	// connectionPool is a static,lazily initialized pool of connections
    val connection = ConnectionPool.getConnection()
	partitionOfRecords.foreach(record => connection.send(record))
	ConnectionPool.returnConnection(connection)  // return to the pool for future reuse	
	}
}
```  

> 需要注意的是，池中的连接对象应该根据需要延迟创建，并且在空闲一段时间后自动超时。这样就获取了最有效的方式发生数据到外部系统。  





 







  
  

 















  
  



 