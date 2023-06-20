---
layout: post
title: SpringCloudAlibaba(六):SpringCloudAlibaba Sentinel实现限流降级
date: 2020-05-20 00:00:00 +0300
description: SpringCloudAlibaba(六)
img: springcloudalibaba/5/dubbo_white.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
大概有半个月没有进行更新了，确实也在纠结一些SpringCloudAlibaba的使用场景问题。同时基于当前的业务与人员配置来考虑甚至有点想放弃微服务的方案了。  
Sentinel的限流降级是必不可少的场景，其实也是基于当前的业务考虑是否需要Sentinel。   
*以上的考虑核心问题就是人员配置的问题，当前我们负责该项目的人员较少，资源较少。所以才有所感*  
但是最终肯定是需要Sentinel的场景的，还是直接一步到位吧!  
![](https://sentinelguard.io/img/sentinel_colorful.png)    

### Setinel的基本概念与使用场景
Setinel的介绍为**一个高可用的流量控制与防护组件**。 流量控制与流量防护就可以看到这个组件的意义就是为了保障微服务的稳定性。Sentinel与原油SpringCloud家族的Hystrix的意义是一致的。  
其实能够实现该方案的场景很多，例如一开始提到的本来没有准备使用Sentinel一个是因为业务的原因。另外一个就是我们本身的一些问题从而考虑使用一些更简单的方案来实现。例如 **nginx** 等。  
![](http://nginx.org/nginx.png)  
当然相对于nginx来说，Sentinel能够实现的功能与灵活性更好一些。Sentinel的功能更多，所以我也会慢慢来开始Sentinel的使用介绍。首先我们使用Sentinel实现一个限流的功能。当然首先我们需要一套Sentinel的环境。

####    部署Sentinel Dashboard 
Sentinel分为两个部分，sentinel-core与sentinel-dashboard。   
sentinel-core 部分能够支持在本地引入sentinel-core进行限流规则的整合与配置。  
sentinel-dashboard 则在core之上能够支持在线的流控规则与熔断规则的维护与调整等。  
言归正传我们先部署一个Sentinel Dashboard。  
*   项目地址：https://github.com/alibaba/Sentinel
*   下载地址: https://github.com/alibaba/Sentinel/releases
本次我们选择当前的最新版 v1.7.2进行部署。Sentinel使用的SpringBoot进行的开发，所以直接下载jar包启动即可使用。
```shell script
java -jar sentinel-dashboard-1.7.2.jar
```
由于Sentinel-Dashboard是使用SpringBoot进行开发的，所以本身没有太多的配置文件。默认的端口为8080。如果端口冲突可以使用 **--server.port** 进行修改绑定。启动成功后使用浏览器访问可以看到如下页面:

![]({{site.baseurl}}/assets/img/springcloudalibaba/6/sentinel_dashboard_login.jpg)  
访问方式为: **sentinel**/**sentinel**     
**WARN:** 如果需要调整相关的参数可以参考github中的具体配置文件进行修改。配置如下:
```properties
#spring settings
spring.http.encoding.force=true
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true

#cookie name setting
server.servlet.session.cookie.name=sentinel_dashboard_cookie

#logging settings
logging.level.org.springframework.web=INFO
logging.file=${user.home}/logs/csp/sentinel-dashboard.log
logging.pattern.file= %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
#logging.pattern.console= %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

#auth settings
auth.filter.exclude-urls=/,/auth/login,/auth/logout,/registry/machine,/version
auth.filter.exclude-url-suffixes=htm,html,js,css,map,ico,ttf,woff,png
# If auth.enabled=false, Sentinel console disable login
auth.username=sentinel
auth.password=sentinel

# Inject the dashboard version. It's required to enable
# filtering in pom.xml for this resource file.
sentinel.dashboard.version=${project.version}
```

####   项目集成
*   首先引入相关的依赖,引入**spring-cloud-starter-alibaba-sentinel**模块。  
    [**groupId**再次提醒一下，com.alibaba.cloud为毕业版本]
```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
*   然后我们就可以把项目中的增加 sentinel-dashboard的相关配置
```properties
spring.cloud.sentinel.transport.dashboard=localhost:8080
```
*   创建启动类与增加Rest访问接口  
**SentinelFlowControlApplication.java**
```java
@SpringBootApplication
public class SentinelFlowControlApplication {

    public static void main(String[] args) {
        SpringApplication.run(SentinelFlowControlApplication.class,args);
    }

}
```
**SentinelTestController.java**  
```java
@RestController
@RequestMapping(value = "sentinel")
public class SentinelTestController {

    @RequestMapping(value = "hello",method = RequestMethod.GET)
    public String hello(){
        return "CainGao";
    }

}
```
*   访问REST接口打开Sentinel-dashboard页面
![]({{site.baseurl}}/assets/img/springcloudalibaba/6/sentinel_dashboard_monitor.jpg)
可以看到我们每次访问的都会在Dashboard中显示出来，并且我们的服务也已经注册到了sentinel中。

*   增加流控规则   
现在我们可以直接在左侧的 簇点链路 中查找到我们访问的端口进行流控规则的设置。 
![]({{site.baseurl}}/assets/img/springcloudalibaba/6/sentinel_dashboard_setting.png)  
现在新增一个 QPS=1 的规则，其他使用默认
![]({{site.baseurl}}/assets/img/springcloudalibaba/6/sentinel_dashboard_add_flow.png)     
配置完成后我们当前的限流效果应该就是每秒只能有一条请求成功发送。其他请求将会直接失败。  

*   直接快速访问接口实现测试
![]({{site.baseurl}}/assets/img/springcloudalibaba/6/sentinel_dashboard_req.png)  
当前可以发现，当请求的QPS大于1时，也就是每秒发送的次数大于1就会直接返回 **Blocked by Sentinel**。  

**本篇源码示例:**
```
https://github.com/CainGao/SpringCloudAlibabaExample  
```