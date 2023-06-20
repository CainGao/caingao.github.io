---
layout: post
title: SkyWalking:分布式架构链路追踪-SkyWalking使用
date: 2020-06-22 00:00:00 +0300
description: SkyWalking分布式链路追踪的使用
img: mac.jpg
tags: [SkyWalking] # add tag
---
### 为什么你需要链路追踪
前面几篇文章提到了微服务相关系统的使用与搭建，在微服务架构下的问题也比较突出。正常系统下我们的每个请求都会在同一个系统中进行输出。
但是在微服务架构中一个请求可能设置一到多个服务进行处理。服务之间相互依赖，服务之间形成一个调用链。如果调用链之间的某个服务出现故障那么整个调用链都将会受到影响。  

架构设计之初就提出了需要进行分布式链路追踪系统，而且当时也对需求进行了大概的一个推演。我们希望能够得到的是一个下图这样的结构。每次请求能够获取到该请求的调用链。
![]({{site.baseurl}}/assets/img/springcloudalibaba/8/req_link.png)

当然上图是一个正常的情况下的请求，异常情况下我们应该获得的是一个能够直接看到异常服务的状态。  
![]({{site.baseurl}}/assets/img/springcloudalibaba/8/req_link_error.png)

### SkyWalking
面对这些情况，我们需要一个能够支撑起该需求的APM工具。目前主要的一些APM工具有,Cat,Zipkin,Pinpoint,SkyWalking。Zipkin是Twitter开源的，Pinpoint是韩国人开源的。Cat与SkyWalking均为国人开发的。所以在选择的时候主要关注的就是国人开发的.(英文不咋滴，怕看不懂文档..)   
其实也大概的翻阅了一下相关的博客，得到了一相关选型的分析与各个工具之间的区别。做了一些排除项，最终选择为SkyWalking。
1.  不要代码侵入(已经上线了几个服务，不想在回去改代码)
2.  分析粒度尽量细
3.  支持较为丰富  

所以今天主要来看一下SkyWalking。  
SkyWalking当前的最新版本已经到了8，我已经再生产环境搭建好了。可以先看一下效果。
*   服务拓扑
![]({{site.baseurl}}/assets/img/springcloudalibaba/8/tract_link.png)

*   请求追踪
![]({{site.baseurl}}/assets/img/springcloudalibaba/8/req_tract_link.png)

可以看到当前的服务调用链。用户发起请求后就会基于调用的相关服务生成调用链拓扑图。而每个请求也能看到详细的调用信息。同时调用拓扑中也除了服务之外也包含对于数据库，外部请求，消息队列等进行拓扑。  

SkyWalking的核心是数据分析与度量的平台，通过Http或者gRPC的方式向信息搜集器(SkyWalking Collecter)上报收集到的客户端采集的信息。信息搜集器(SkyWalking Collecter)对搜集到的结果进行分析与聚合。它的数据主要使用ElasticSearch，MySql，H2，TiDB等进行存储。当然任选其一即可。我们通过UI进行查看分析的数据结果。采集器则负责搜集数据，支持较多的语言 Java，PHP，.Net Core,NodeJS,Golang等。  

### 总结
SkyWalking满足我们的当前需求，最直观的可以通过SkyWalking看到服务调用链是否合理。是不是一个DAG。同时能够分析每个请求的追踪是否有异常。而且支持MQ，MySQL，Http请求等各种方式能够获取到发生异常的点与RT较高的点进行优化。  