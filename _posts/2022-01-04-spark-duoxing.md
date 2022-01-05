---
layout:     post
title:      hue 提交 spark 读取hdfs数据显示为空
subtitle:   
date:       2022-01-04
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据问题型记录
---

申请了大数据中心资源后，拟定使用计算引擎：spark。首先，使用spark读取hdfs数据，出现错误：
1. DFSClient: Failed to connect to /192.168.137.2:50010 for block, add to deadNodes and continue. java.net.ConnectException: Connection timed out: no further information
   java.net.ConnectException: Connection timed out: no further information
