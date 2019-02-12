---
title: docker-compose部署开发环境-Redis、MySQL、RabbitMQ
categories: 
- 自动化构建
tags: 
- devops
---

### 环境内容
Redis、MySQL、RabbitMQ
### 版本选择

软件包 | 	镜像版本
---|---
Redis | row 1 col 2
MySQL | row 2 col 2
RabbitMQ | row 2 col 2

### 文件目录

├── Dockerfile.redis

├── conf

│   └── redis.conf

└── docker-compose.yaml

### Dockerfile.redis


```
FROM redis:4.0.7-alpine

MAINTAINER "sxp" <1013580089@qq.com>

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& mkdir -p /home/redis/conf

COPY ./conf/redis.conf /home/redis/conf/redis.conf

CMD [ "redis-server", "/home/redis/conf/redis.conf" ]
```
### redis.conf


```
# redis 端口设置
port 6379 
# 配置数据持久化
appendonly yes
# 密码
requirepass 123456
# 发布key事件，使用过期事件（当每一个key失效时，都会生成该事件）。
notify-keyspace-events Ex
```

### docker-compose.yaml


```
# 定义docker-compose版本，有些参数只有高版本才有，低版本没有
version: '3.0'
# 定义服务，可以是一个，也可以是多个
services:
	# 具体的服务名，其他的服务可以用过这个名字访问
    mysql:
    	# 定义这个服务所使用的镜像以及镜像版本，若不指定版本默认是latest，生产环境不建议使用latest
        image: hub.c.163.com/yb7553/mysql:latest
        restart: always
        container_name: vic-mysql
        # 环境变量设置，key=value
        environment:
            MYSQL_ROOT_PASSWORD: 123456
        # 定义暴露端口，若写成 '3306:3306/tcp' 的格式则表示指定宿主机的80转发到容器的3306，若写成 '3306/tcp' 则表示Docker将随机分配一个宿主机端口给容器内的3306端口，tcp表示协议类型，也可以是udp
        ports:
            - "3306:3306/tcp"
        # 网络定义部分
        networks:
        	# 定义这个容器运行在哪个虚拟网络里面
            vic:
            	# 设置这个容器在网络中的别名，可以是一个，可以是多个，其他的服务可以用过这个名字访问
                aliases:
                    - mysql
        # 定义卷，可以是卷，也可以是目录，可以设置容器内的权限是什么
        volumes:
        	# 格式：src:dest:mode
        	# src: 可以是卷名，宿主机目录等，可以是文件或者目录，若是文件则文件必须存在，否则会是目录的形式挂载
        	# dest: 容器内的路径，可以是文件可以是目录，若是文件则文件必须存在，否则会是目录的形式挂载
        	# mode: 权限，只读(ro)，可读可写(rw)
            - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
    rabbitmq:
        image: rabbitmq:3.7.3-management-alpine
        container_name: vic-rabbitmq
        ports:
            - "5672:5672"
            - "15672:15672"
        environment:
            RABBITMQ_DEFAULT_USER: guest
            RABBITMQ_DEFAULT_PASS: guest
        networks:
            vic:
                aliases:
                    - rabbitmq
        volumes:
            - ./mysql:/var/lib/mysql
            - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
    redis:
        image: hub.c.163.com/yb7553/redis:latest
        build:
            context: .
            dockerfile: Dockerfile.redis
        container_name: vic-redis_single
        ports:
            - "6379:6379"
        networks:
            vic:
                aliases:
                    - redis
        volumes:
            - ./data:/redis-data:rw
# 定义这个compose文件中使用的网络
networks:
	# 网络名称，和上文中的pig一致。
    vic:
    	# 是否是外部网络，理解为如果是外部网络那么网络的名字就叫pig，这样方便其他的服务接入到这个网络，如果不是，在你使用docker-compose up -d 命令的时候他会以你当前的文件夹的名字加上这里定义的网络名作为你这个compose的网络。
        external: true
```
### 启动


```
# 生成网络
docker network create vic
# 查看网络
[root@localhost compose]# docker network list
NETWORK ID          NAME                   DRIVER              SCOPE
d43944f3cb12        pig                    bridge              local
# 启动
[root@localhost compose]# docker-compose up -d
```
