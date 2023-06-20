---
layout: post
title: SpringCloudAlibaba(七):SpringCloudAlibaba Sentinel使用nacos配置
date: 2020-05-20 00:00:00 +0300
description: SpringCloudAlibaba(七)
img: springcloudalibaba/5/dubbo_white.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
前一篇文章中我们已经完成了Sentinel限流的一个程序，通过Dashboard进行Sentinel的限流配置。 不知道大家有没有发现我们在使用Sentinel Dashboard的时候它本身是没有数据库的。 
所以它在重启之后就会丢失我们配置的限流等规则。所以我们需要把限流的规则持久化一下。  
在之前我们都会通过配置文件来管理相关的配置信息，但是我们之前已经用Nacos实现了集中化的配置管理。所以，这次我们也同样使用Nacos来实现Sentinel规则的配置管理。  

### Nacos管理Sentinel限流规则
我们本次会用到Nacos与SentinelDashboard，所以首先保证两个服务都处于启动状态。  
####   集成依赖
#####   pom修改配置
```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.csp</groupId>
        <artifactId>sentinel-datasource-nacos</artifactId>
    </dependency>
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
对比上一篇增加了nacos的datasource。用以支持在nacos中配置限流规则。
#####   修改配置文件application.properties
```properties
server.port=8999
spring.application.name=sentinel-nacos-conf
#   sentinel.dashboard配置
spring.cloud.sentinel.transport.dashboard=localhost:8080
#   nacos配置
spring.cloud.sentinel.datasource.flow.nacos.server-addr=localhost:8848
#   dataid
spring.cloud.sentinel.datasource.flow.nacos.data-id=${spring.application.name}-sentinel-flow
spring.cloud.sentinel.datasource.flow.nacos.group-id=DEFAULT_GROUP
#   定义存储规则的枚举
spring.cloud.sentinel.datasource.flow.nacos.rule-type=flow
```
#####   创建启动类与Rest接口
*   SentinelNacosConfApplication.java
```java
@SpringBootApplication
public class SentinelNacosConfApplication {

    public static void main(String[] args) {
        SpringApplication.run(SentinelNacosConfApplication.class,args);
    }

}
```
*   SentinelHelloController.java
```java
@RestController
@RequestMapping(value = "sentinel")
public class SentinelHelloController {
    private Logger LOG = LoggerFactory.getLogger(this.getClass());

    @RequestMapping(value = "hello",method = RequestMethod.GET)
    public String hello(){
        return "CainGao";
    }

}
```

####    Nacos配置
#####   Nacos增加Sentinel配置文件
nacos增加配置，基于 dataId 可以看到需要增加的配置名称为 sentinel-nacos-conf-sentinel-flow。配置内容为：
```json
[
    {
        "resource": "/sentinel/hello",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```
参数|解释
---|---
resource|表示需要进行限流的资源
limitApp|流控针对的来源，default表示不区分来源
grade|限流阈值类型（QPS：0，并发线程数：1）
count|限流的阈值
strategy|判断根据资源本身还是其他资源，默认为资源本身
controlBehavior|拦截后的策略(0:直接拒绝/1:排队等待/2:匀速等待)
clusterModel|是否为集群模式  
![]({{site.baseurl}}/assets/img/springcloudalibaba/7/sentinel_nacos_conf.png)  
设置完成后我们启动应用，当应用启动完成后进入Sentinel控制台就会发现规则已经设置到了Sentinel Dashboard中。
![]({{site.baseurl}}/assets/img/springcloudalibaba/7/sentinel_nacos_sync.png)    
可以看到流控规则已经写入成功。

#####   进行应用测试
![]({{site.baseurl}}/assets/img/springcloudalibaba/7/sentinel_rest_test.png)


####    总结
我们可以通过Nacos进行动态的管理Sentinel的各种规则，毕竟基于我们的场景来说更加希望配置能够统一的进行管理。当然各种前期的约定就需要设计很多了。例如各种策略的配置名称，毕竟是希望能够让所有人都能直接的基于约定来实现配置。不要出现 例如 我设置为 {spring.application.name}-sentinel-flow,另外一个同事直接设置为 {spring.application.name}-flow。
