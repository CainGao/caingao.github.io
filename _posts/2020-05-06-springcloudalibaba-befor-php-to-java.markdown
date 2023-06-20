---
layout: post
title: SpringCloudAlibaba(前传)：接到个任务,php转Java
date: 2020-05-06 00:00:00 +0300
description: SpringCloudAlibaba(前传)
img: springcloudalibaba/Spring.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---

近期手头上的工作差不多完成了，可能作为数据开发来说最近的数据已足以支撑当前业务所以没有较多的工作安排。所以突然收到一个任务，php转Java。别误会，不是说我的开发语言，是企业的开发语言要从php转到Java。而企业内主要语言是Java的有我跟另外以为老哥，而我近期的工作基本完成，这个伟大的任务就落到了我的头上。


### 脱胎换骨的升级-更换开发语言
可能有人见过这句话,这句话是<阿里技术这十年>上写关于淘宝网从php转换为Java开发语言时书中用做菜单的一句话。04年淘宝网完成了php到Java语言的过渡，操刀者是Sun公司的第一流专家。16年后有一家公司，也要进行php到Java语言的过渡，操刀者是个新晋菜鸟。  
![]({{site.baseurl}}/assets/img/springcloudalibaba/befor/java_vs_php.png)  
### 为什么要用Java替换PHP?
首先,我不是一个PHP的开发者，而且更多的工作其实在数据上。ETL、数据分析、实时计算、离线计算等工作。所以对于php替换到Java我也难以说出个1 2 3 来。可能近期遇到的一些问题是让我们选择替换的原因吧。
*   没有php sdk?  
    对接某云厂商的产品,提供了python,Java等相关语言的sdk,但是没有php的sdk。最终方案，使用Java做个gateway是解释php的请求。  
*   性能问题  
    随着业务的发展并发数量也越来越高，php的并发性能压力也就越来越大。同时社区也没有教好的对于该问题的支持。  
*   生态问题  
    生态问题其实跟第一个问题是一样的，据说php的相关生态内容比之于Java欠缺较多。特别是当前都在微服务的场景下，对于php来说不是很友好的。  
*   规范问题   
    PHP开发不像是Java有严格的编码规范，每个人一套自己的风格，api管理也较为混乱。所以在代码的维护上非常的麻烦，同时php不需要进行编译。在开发时可能是优点，但是在企业应用开发中可能就是缺点。无法在早期直接看到错误。      
    其他...

### 为什么选择Java?
当前来实现后端开发的语言较多。Java、PHP、Python和Go等都有一批的拥护者，在当前的环境下常用的语言 PHP、Java、Python、Scala是公司的常用语言。其中PHP占的份额最高实现了所有的Web相关的功能。Java基本实现了一些gateway或者某些特殊的数据处理或其他项目，Python主要应用与运维、Scala则是大数据处理的常用语言。    
主要考虑内部成员的语言掌握情况与社区的活跃情况下，我们选择了Java作为PHP的替代语言。 

### SpringCloud Alibaba
其实在做选型的时候是带有一些需求的。所以在做选型的时候因为这些简单的需求倾向直接就选型了SpringCloudAlibaba。
1.  配置中心&注册中心
2.  分布式链路跟踪
3.  RPC  

其实需求比较简单，但是都是经过一次次的‘灾难’取得的教训:  
1.  **分布式链路跟踪是客户的请求到最终的一个错误整体链路查询所get到的**   
2.  **配置中心是某些大哥把配置错误的提交到了github。导致...**     
3.  **RPC较为简单，仅仅是想要把后台的http请求替换为rpc**

可选的方案可能很多，Dubbo，SpringCloud。SpringCloudAlibaba。基于内部的情况，我们选型SpringCloudAlibaba。   

跟随阿里的脚步在国内的开发者圈子中是个主流，Dubbo开源后持续性的火了很久。但是后来停止维护一段时间。SpringCloud生态也是较为强大的。SpringCloudAlibaba可以说是整合了 Dubbo 的RPC与 SpringCloud的生态优势实现的。同时SpringCloudAlibaba的相关组件 Nacos与 sleuth 实现了配置&注册中心和分布式链路跟踪的需求。 Seata的分布式事物中间件也会成为主要的使用场景。Sentinel实现分布式系统的流控。  
### 总结
Java用来替换PHP 我可能没有什么发言权，只是一个任务的执行者。毕竟我不是从PHP的高并发环境下走到Java的微服务生态中。所以，总结的可能不是很完美。如果各位有任何其他的想法或者对于Java框架的选型问题可以私信我，一起谈谈PHP转换Java语言的问题。  微信公众号:指尖数虫 
