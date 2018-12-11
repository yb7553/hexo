---
title: 网络结构图
date: 2018-11-29 13:55:05
categories: 
- 基础设施
tags: 
- base
---
主流的设计一般会采用微服务架构。其思路不是开发一个巨大的单体式应用，而是将应用分解为小的、互相连接的微服务。一个微服务完成某个特定功能，比如乘客管理和下单管理等。每个微服务都有自己的业务逻辑和适配器。一些微服务还会提供API接口给其他微服务和应用客户端使用。
<!-- more -->
![image.png](https://upload-images.jianshu.io/upload_images/5189695-4c13ea4c090562dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

