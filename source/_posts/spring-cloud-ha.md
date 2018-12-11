---
title: 微服务高可用
date: 2018-11-28 12:55:03
categories: 
- 微服务
tags: 
- spring cloud
---
动态的环境和分布式的系统，比如微服务，它们出现故障的几率更大；发生故障的服务应该被隔离开来，实现优雅的服务降级，提升用户体验；70% 的故障都是因为代码变更引起的，所以有时候回退代码并不算是什么坏事；如果发生故障，就要让它们快速而独立的发生；一个团队无法控制他们服务的依赖项；缓存、隔板、回路断路器和速率限定器这些架构模式有助于构建可靠的微服务。
<!-- more -->
![image.png](https://upload-images.jianshu.io/upload_images/5189695-9db6a584bacc7efa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
