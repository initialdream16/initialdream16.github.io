---
layout:     post
title:      mkdocs
subtitle:   快速、简单、华丽的静态网站生成器,适用于构建项目文档
date:       2021-12-31
author:     果然
header-img: img/timg.jpg
catalog: true
tags:
    - mkdocs
---


以下是对 mkdocs([中文文档](https://mkdocs-material.zimoapps.com/)) 的 docker 部署进行说明。由首页的说明，我们知道 mkdocs 在撰写文档的过程中可以进行实时预览，前提是我们开启命令 `mkdocs serve`。

下面尝试将 **mkdocs** 部署至 **docker** 和 **github** 中。

尝试使用 Dockerfile 搭建镜像和 生成项目指令两种方法进行部署实践。其实，应该说明是一种方法，因为二者的原理是一致的，区别仅仅在 docker 指令是否写在 Dockerfile 文件中。

在部署之前，我们先拉取 material 主题的 mkdocs。拉取方法很简单：  
<center>docker pull squidfunk/mkdocs-material</center>   

# 部署至 docker
## Dockerfile

首先，在本地计算机上测试静态站点页面可访问。 
其次，将项目目录打包为 tar.gz 文件。简单说明下，打包软件使用 gzip。这里，我们的项目结果如下：    

```
test_mkdocs  
	mymkdocs  
		docs/        # docs 文档主目录
			index.md  
			detailpage/  
				docker.md  
			testpage/  
				mockdocs_practics.md  
				mkdocs_docker_Dockerfile.md
		mkdocs.yml       # mkdocs 的 yml 配置文件  
```  

打包为 tar.gz 的过程：右键首先保存为 tar 格式，后再添加格式为 gzip 的压缩包。完成后，可以在 ubuntu/centos 上测试解压：  
<center>tar -xvzf test_mkdocs</center> 

第三，在虚拟机上随机创建文件夹，后续的操作在创建的文件夹路径下，并编写 Dockerfile 文件，放入该路径下：  

```
mkdir -p /usr/local/demo  
cd /usr/local/demo        # 路径中包含 Dockerfile 和压缩包  
vim Dockerfile  
	FROM squidfunk/mkdocs-material
	MAINTAINER guomiao@guomiao@163.com
	WORKDIR /docs/test_mkdocs/mymkdocs  
	ADD test_mkdocs.tar.gz /docs

docker build -t mymy-mkdocs .  
docker images  
docker run -dit -p 9998:8000 mymy-mkdocs  
```  

![search](img/3.png) 

对 Dockerfile 文件进行说明的 docker 的基础命令进行说明，如下：  
* <font color=red>FROM</font>  指定基础镜像。  
* <font color=red>WORKDIR</font> 用于指定容器的一个目录，容器启动时执行的命令会在该目录下执行。部署 mkdocs 的路径需要编写正确(结合解压路径)，否则会出错 mkdocs.yml 找不到的。    
* <font color=red>ADD</font> 拷贝宿主机文件至docker容器下的文件路径。  
* 此外，还使用到 [<font color=red>-v</font>](https://blog.csdn.net/hnmpf/article/details/80924494) 、[<font color=red>docker inspect</font>](https://www.cnblogs.com/gcgc/p/10831711.html)、[<font color=red>进入容器</font>](https://blog.csdn.net/weixin_43448760/article/details/104427609)。  

## docker 指令  
生成项目目录，  
*`docker run -it --rm -v ~/docs:/docs squidfunk/mkdocs-material new my-project`*   
*`sudo chmod -R 777 /xxx/`*  
运行起来，  
*`docker run -it --name mkdocs --rm -v ~/docs:/docs -p 58000:8000 --workdir /docs/my-project squidfunk/mkdocs-material serve -a 0.0.0.0:8000`*  

# 部署至 github  
主要使用代码 *mkdocs gh-deploy* 。详细可见首页 - 其他的说明。

每日一图(图来源于网络)，哈哈。
  
![fengjing](img/4.jpeg)  

  

