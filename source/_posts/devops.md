---
title: docker基础架构
---
![image.png](https://upload-images.jianshu.io/upload_images/5189695-0f77a07c50f709a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# Docker
## 一、docker基础概念
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

#### 1、Docker有以下几个部分组成：
	
```
- Client客户端
- Docker Daemon守护进程
- Docker Image镜像
- DockerContainer容器
- Registry镜像仓库
```


#### 2、Docker的三大核心概念：镜像、容器、仓库

```
- 镜像：类似虚拟机的镜像、用俗话说就是安装文件。
- 容器：类似一个轻量级的沙箱，容器是从镜像创建应用运行实例，
- 	      可以将其启动、开始、停止、删除、而这些容器都是相互隔离、互不可见的。
- 仓库：类似代码仓库，是Docker集中存放镜像文件的场所。
```

 
Docker采用 C/S架构 Docker daemon 作为服务端接受来自客户的请求，
并处理这些请求（创建、运行、分发容器）。 
客户端和服务端既可以运行在一个机器上，也可通过 socket 或者RESTful API 来进行通信。

## 二.docker安装
linux系统环境 ，同时需要注意Linux系统必须能够上网,虚拟机使用桥接模式
建议使用centos7版本安装docker，docker容器需要镜像文件容器与镜像文件的关系就类似于类与对象的关系。

## 三.docker安装步骤
	
```
1、使用yum命令在线安装:yum install docker
	2、查看docker的版本命令:docker -v
	3、启动docker：systemctl start docker
	4、停止docker：systemctl stop docker
	5、重启docker：systemctl restart docker
	6、查看docker状态：systemctl status docker
	7、开机启动：systemctl enable docker
	8、查看docker概要信息：docker info
	9、查看docker帮助文档：docker --help
```


    systemctl命令是系统服务管理器指令，它是 service 和 chkconfig 两个命令组合



## 四、docker操作镜像
   
```
1、docker images #查看本机上所有的镜像文件
    REPOSITORY：镜像所在的仓库名称
	TAG：镜像标签
	IMAGE ID：镜像ID
	CREATED：镜像的创建日期（不是获取该镜像的日期）
	SIZE：镜像大小
```


  
```
2、docker search 镜像名称 #在注册中心(仓库)上进行搜索镜像
	NAME：仓库名称
	DESCRIPTION：镜像描述
	STARS：用户评价，反应一个镜像的受欢迎程度
	OFFICIAL：是否官方
    AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的
```

 
```
3、下载镜像:docker pull zookeeper
   	注意:默认是从官网下载
	可以ustc的docker镜像加速器速度很快。
	ustc docker mirror的优势之一就是不需要注册，是真正的公共服务。
	(1)编辑该文件：vi /etc/docker/daemon.json  // 如果该文件不存在就手动创建；说明：在centos7.x下，通过vi。
	(2)在该文件中输入如下内容：
		{
		"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
		}
	(3)注意：一定要重启docker服务(systemctl restart docker.service)，如果重启docker后无法加速，可以重新启动OS

	
	https://lug.ustc.edu.cn/wiki/mirrors/help/docker
```


```
4、docker rmi $IMAGE_ID：删除指定镜像
    docker rmi `docker images -q`：删除所有镜像
    #$IMAGE_ID 镜像ID，通过docker images可以查看到镜像的ID
```

## 五、docker容器操作

```
1.查看容器操作:
	1)查看正在运行的容器:docker ps
	2)查看所有的容器（启动过的历史容器）:docker ps -a
	3)查看最后一次运行的容器：docker ps -l
	4)查看停止的容器：docker ps -f status=exited
```



```
2.创建与启动容器
(1)创建容器需要的参数
创建容器命令：docker run
	-i：表示运行容器
	-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
	--name :为创建的容器命名。比如:--name=mysqlcat
	-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），
	      可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
	-d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，
	      如果只加-i -t两个参数，创建后就会自动进去容器）。
	-p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个－p做多个端口映射
(2)创建容器的两种方式:
	交互式容器：创建容器启动后会进入其命令行。
		docker -it --name=mycentos centos:7 /bin/bash
		说明:centos:7----可以使用docker images查看 7表示TAG：镜像标签 如果镜像名称为latest，就可以不用写
		/bin/bash表示linux的执行命令
	注意: 使用exit命令 退出当前容器，容器会关机

	守护式容器:类似于启动服务的进程一样，不会进入容器
		docker -di --name=mycentos2 centos:7
		进入容器:docke exec -it mycentos2 /bin/bash
	注意:使用exit命令，退出当前容器，但是容器不会关机
(3)停止容器和启动容器
	停止容器:docker stop 容器的名称/容器的id
	启动容器:docker start 容器的名称/容器的id
	说明:容器的id可以通过docker ps -a 查看到，也就是:CONTAINER ID

(4)删除容器
    删除指定的容器：docker rm $CONTAINER_ID/NAME 注意:只能删除停掉的容器
    删除所有容器：docker rm `docker ps -a -q`	

(5)文件拷贝(从宿主机拷贝到我们创建的容器)
	docker cp 需要拷贝的文件或者目录 容器名称:容器的目录
	注意:容器名称后面有冒号

	也可以将文件从容器内拷贝出来
	docker cp 容器名称:容器目录 需要拷贝的文件或目录

(6)目录挂载
    我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，
这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。
创建容器 添加-v参数 后边为   宿主机目录:容器目录
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos2 centos:7
如果你共享的是多级的目录，可能会出现权限不足的提示。
说明:容器里的目录可以不存在在进行映射的时候会自动创建。

这是因为CentOS7中的安全模块selinux把权限禁掉了，
我们在创建容器的时候需要添加参数  --privileged=true  来解决挂载的目录没有权限的问题
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos --privileged=true centos:7

(7)查看容器IP地址
    我们可以通过以下命令查看容器运行的各种数据
docker inspect 容器的名称

也可以直接执行下面的命令直接输出IP地址
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycentos2
```

