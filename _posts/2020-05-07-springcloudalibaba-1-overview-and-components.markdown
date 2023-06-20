---
layout: post
title: SpringCloudAlibaba(一)：概述与重要组件
date: 2020-05-07 00:00:00 +0300
description: SpringCloudAlibaba(一)
img: springcloudalibaba/Spring.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
前一篇提到了我们为什么要替换PHP语言采用Java语言。而Java语言的框架选型上来说有太多的选择，常见的有Dubbo，SpringCloud等。我们选择了SpringCloudAlibaba。替换PHP语言到SpringCloudAlibaba是个大工程，主要是业务迁移部分。讨论之初我也确认过是否迁移原有的业务，得到的明确答复是 **迁移**。  那么这么来说也就简单了，复杂的就是工期问题了。  

### SpringCloudAlibaba是什么？
**SpringCloud Alibaba是Alibaba结合自身的微服务实践开源的一套微服务全家桶**，在SpringCloud项目中进行孵化并且毕业。既然是SpringCloud的项目那么阿里云其实包含其商业化的产品。 例如Nacos在阿里云就有其商业化的版本 **MSE**。 同时SpringCloud Alibaba的相关组件是经历过双十一大促考验的产品。稳定性较高。

### SpringCloud Alibaba与SpringCloud
SpringCloud Alibaba是SpringCloud的子项目，其实很多相关的文章都提到了SpringCloud Alibaba与SpringCloud的关系，其中有很多的论点都比较有意思。大家可以去搜索一下。  
SpringCloud Alibaba是依赖SpringCloud相关的标准实现的一套微服务的架构。结合阿里巴巴的相关实践与阿里云的相关服务实现的一些组件得以更快的实现相关产品业务。


### SpringCloud Alibaba主要功能
*   **分布式配置**  
    分布式系统的外部配置管理，配置中心可视化、分环境配置控制。配置动态更新能力。  
*   **服务注册与发现**  
    适配SpringCloud标准的服务注册与服务发现管理。
*   **服务限流与降级**  
    可通过控制台进行实时的修改限流降级的规则，实时的Metrics监控。支持多种协议
*   **消息驱动**  
    基于RocketMQ实现消息驱动的业务场景开发。  
*   **分布式事物**
    开源Seata使用@GlobalTransactional注解，零侵入的实现分布式事物的支持。  
    
### SpringCloud Alibaba核心组件

####    Nacos (配置中心与服务注册与发现)
Nacos实现了服务的配置中心与服务注册发现的功能，Nacos可以通过可视化的配置降低相关的学习与维护成本，实现动态的配置管理与分环境的配置中心控制。 同时Nacos提供了基于http/RCP的服务注册与发现功能。  

####    Sentinel (分布式流控)
Sentinel是面向分布式微服务架构的轻量级高可用的流控组件，以流量作为切入点，从流量控制，熔断降级，系统负载保护等维度帮助用户保证服务的稳定性。常用与实现限流、熔断降级等策略。  

####    RocketMQ (消息队列)
RocketMQ基于Java的高性能、高吞吐量的消息队列，在SpringCloud Alibaba生态用于实现消息驱动的业务开发，常见的消息队列有Kafka、RocketMQ、RabbitMQ等，相关的比较文档可以自行去翻阅。

####    Seata  (分布式事物)
既然是微服务的产品，那么肯定会用到分布式事物。Seata就是阿里巴巴开源的一个高性能分布式事物的解决方案。

####    Dubbo   (RPC)
Dubbo已经在圈内很火了,SpringCloud Alibaba基于上面提到的Nacos服务注册中心也同样整合了Dubbo。 

####    其他
SpringCloud Alibaba还有一些其他的组件选择，例如schedulerX、SMS、OSS等。但是由于其主要是阿里云的商业化产品就不再过多的进行介绍。集成其商业化产品时才能用到。  

### 总结
SpringCloud Alibaba是基于SpringCloud标准由阿里巴巴实现的微服务全家桶，可插拔的方式实现组件的替换，在某些场景中我们需要的组件可以自由进行选择。例如需要分布式链路跟踪我们可以增加sleuth组件用于实现分布式链路跟踪业务等。  
很多人提到SpringCloudAlibaba的商业问题，记得当年SpringCloudAlibaba推出第一版的时候我也评论了...卖产品全家桶。不可否认是有那么一些，但是其实它本身的很多组件又不一定非要选择商业版本。这个可以自由进行选择。  
