---
title: elk日志平台架构
date: 2018-11-28 14:28:30
categories: 
- 日志管理
tags: 
- log
---
此外还要注意日志收集的部分。像传统的方法存在一个问题，在 Docker 环境下，一个宿主机上面有很多不同服务的业务实例，那这些日志怎么办？难道要按个做脚本采集文件信息吗？更大的问题是你是有 file 的，如果是云上买的，你可能需要扩大磁盘的容量，因为你有可能需要做监控脚本，清理日志。在做到微服务、容器化之后，第一个要做的就是让你的日志减少，这样就没有文件，直接通过 SDD Out 的方式，对 logstash、sleuth 采集起来之后，可以直接送日志数据，中间不会产生一个日志，不会清理。因为我们之前一开始还没有做这套运维基础的时候，经常都需要手工的清日志，在业务量突增的时候会产生很多日志。我们做征信的时候中间要打很多日志，可能一次请求里面要打十几兆的日志，因为涉及到的第三方数据源实在太多了，中间有很多计算的过程也要打印下来，否则的话没法知道中间到底出了什么问题。
<!-- more -->

![image.png](https://upload-images.jianshu.io/upload_images/5189695-33f92605c263a270.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5189695-dd431d6c098f27df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5189695-59bf65779f817045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

