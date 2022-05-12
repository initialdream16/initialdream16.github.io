---
layout:     post
title:      docker-compose
subtitle:   一个用来定义和运行复杂应用的docker工具。
date:       2022-05-11
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - 大数据组件技术选型
---  
## [compose介绍](https://blog.csdn.net/pushiqiang/article/details/78682323)  
一个使用docker容器的应用，通常由多个容器组成。使用docker compose不再需要使用shell脚本来启动容器。compose通过一个配置文件来管理多个docker容器，通过services来定义所有容器，使用docker-compose脚本来启动、停止和重启应用，非常适合组合使用多个容器进行开发的场景。  
## 安装docker  
Docker从1.13.x版本开始，版本分为企业版EE和社区版CE，版本号也改为按照时间线来发布，比如17.03就是2017年3月。  
安装完成后，执行：*service docker start*命令。  

![docker install test](https://initialdream16.github.io/img/docker-install.png)  

## 安装docker-compose  
使用daocloud上下载docker-compose二进制文件安装：  

![docker install test](https://initialdream16.github.io/img/docker-compose.png)  

执行curl命令后，它会将docker-compose放入*/usr/local/bin*，创建一个软链接*sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose*。  
## [docker-compose 文件结构和示例](https://blog.csdn.net/pushiqiang/article/details/78682323)      
docker-compose.yml  
```
version："3"
services:
	...
networks:
	...
volumes:
	... 
```  
**示例：**通过docker-compose构建一个在docker中运行的基于python flask框架的web应用。  
step1：定义Python应用
* 创建工程目录  
  
![docker install test](https://initialdream16.github.io/img/tree.png)   

* 在src/目录下创建Python flask应用 app.py文件  
* 创建Python需求文件 requirements.txt  
  
step2：创建容器的Dockerfile文件  
step3：定义docker-compose脚本  
step4：使用Compose构建并运行您的应用程序  

![docker install test](https://initialdream16.github.io/img/compose.png)  

step5：编辑compose文件以添加文件绑定挂载  
通过volumes（卷）将主机上的项目目录（compose_test/src）挂载到容器中的/opt/src目录，允许您即时修改代码，而无需重新构建镜像。  

![docker install test](https://initialdream16.github.io/img/volumes.png)  

从运行结果：通过修改app.py文件，无需重启docker-compose，web端自动更新了。   
step6：重新构建和运行应用程序      

最后，展示下status：  

![docker install test](https://initialdream16.github.io/img/status.png)  
















  
  



 
