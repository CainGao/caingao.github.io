---
layout: post
title: SpringCloudAlibaba(五):SpringCloudAlibaba+Dubbo实践
date: 2020-05-20 00:00:00 +0300
description: SpringCloudAlibaba(五)
img: springcloudalibaba/5/dubbo_white.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
SpringCloud与Dubbo两个框架一直以来都被用来作为两种框架进行比较，其实对于两者之间进行比较是不公平的。SpringCloud可以说是一套完整的解决方案而Dubbo其实只是一种RPC和服务治理的实现方案。  

Dubbo一直以来在国内有较多的使用，虽然阿里中间有一段时间没有进行维护，但是受众群体还是比较多的。同时在阿里不再维护的那段时期也由当当维护的DubboX推出。但是Dubbo的相关周边组件也依然不是那么的完善。   

一直以来SpringCloud与Dubbo的整合方案也不是那么的完善，相对来说整合的都比较‘丑’。Dubbo的注册中心是Zookeeper而SpringCloud一开始是不支持Zookeeper作为注册中心的。所以在大部分的公司架构中都是二者之间取其一。  

SpringCloudAlibaba的出现解决了这样的问题，SpringCloudAlibaba与Dubbo都选用Nacos作为服务的注册中心，并且可以类似于传统项目的SpringCloud项目一样使用Feign进行消费。今天就尝试一下使用Dubbo连接Nacos注册中心。

### 开始之前
可能之前写过Dubbo的同学比较了解，Dubbo的服务提供方基本需要支持两个项目，**api**与**provider**。
**api**负责定义服务所提供的相关接口，以供消费方进行依赖。 **provider**则提供相关接口的具体实现。  
例如:   
![]({{site.baseurl}}/assets/img/springcloudalibaba/5/nacos_discovery_dubbo.png)  
以上的结构是遵守了几年的一个规范，提供的相关接口由api项目负责定义，provider负责进行实现，consumer则依赖相关api进行服务的调用。下面进行分步骤的实现。 

### api 定义服务接口
**IHelloService.java**  
```java
public interface IHelloService {

    String hello(String name);

}
```  

### privider构建服务提供方
*   第一步：构建SpringBoot项目，引入SpringCloudAlibaba Nacos Dubbo相关依赖。同时也需要引入刚才的api项目。
```xml
<dependencies>
    <dependency>
        <groupId>top.anydata.products.web</groupId>
        <artifactId>nacos-discovery-dubbo-api</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
    <!--com.alibaba.cloud为毕业版本,org.springfremework.cloud为孵化版本-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
    <!--必须包含,否则会报错-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
*   第二步:实现api项目中定义的接口
**HelloServiceImpl.java**
```java
@Service
public class HelloServiceImpl implements IHelloService {

    @Override
    public String hello(String name) {
        return "Hello:"+name;
    }

}
```
**WARN: 这里的@Service注解不是springframework的而是org.apache.dubbo.config.annotation.Service注解**

*   第三步:配置文件
```yaml
server:
  port: 8080
spring:
  application:
    name: nacos-dubbo-provider
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      discovery:
        enabled: true
        register-enabled: true
        server-addr: localhost:8848
  profiles:
    active: true
dubbo:
  scan:
    base-packages: top.anydata.products.web.example.nacos_discovery_dubbo_provider.service
  protocol:
    name: dubbo
    port: -1
  registry:
    address: spring-cloud://localhost
  application:
    qos-enable: true
  cloud:
    subscribed-services: /
```   
配置nacos注册中心相关，同时增加dubbo相关的配置信息。其中 **scan** 表示要扫描的dubbo基础包。

*   创建SpringBootApplication启动类与启动验证
**NacosDubboProviderApplication.java**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosDubboProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosDubboProviderApplication.class,args);
    }

}
```
**启动**  
![]({{site.baseurl}}/assets/img/springcloudalibaba/5/nacos_discovery_dubbo_provider.png)  
启动成功后在Nacos服务列表中即可看到入上服务。

### consumer服务消费方
####   第一步:构建服务消费调用项目，pom.xml 所需的依赖与 provider一致。
####    第二步：创建Controller进行服务消费
**HelloController.java**
```java
@RestController
public class HelloController {

    @Reference
    IHelloService helloService;

    @RequestMapping(value = "hello")
    public String hello(String name){
        return helloService.hello(name);
    }

}
```
**WARN: @Reference 是org.apache.dubbo.config.annotation.Reference 有较多的参数配置可选,例如check=true校验服务是否健康等**

####    第三步:Dubbo相关配置
```yaml
server:
  port: 8081
  application:
    name: nacos-dubbo-consumer
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      discovery:
        enabled: true
        register-enabled: true
        server-addr: localhost:8848
spring:
  application:
    name: nacos-dubbo-consumer
dubbo:
  registry:
    address: spring-cloud://localhost
  cloud:
    subscribed-services: nacos-dubbo-provider
  application:
    qos-enable: false
```
**WARN:此处 dubbo.cloud.subscribed-services为调用的服务,配置为服务的注册名称。可以在Nacos中查看**

####    第四步:创建**NacosDubboConsumerApplication**类并启动消费
**NacosDubboConsumerApplication.java**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosDubboConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosDubboConsumerApplication.class,args);
    }

}
```
启动成功后仍然使用postman进行测试。  
![]({{site.baseurl}}/assets/img/springcloudalibaba/5/naocs_discovery_dubbo_consumer.png)  

### 结论
通过以上的测试，可以看到SpringCloud与Dubbo使用Nacos注册中心来实现还是非常简单的，Dubbo 的RPC性能优势还是非常重要的。同时仍可以享受SpringCloud的各种便利。SpringCloudAlibaba提供的整合方案能够融合SpringCloud与Dubbo的各种优势，同时也便于原有SpringCloud与Dubbo用户的迁移。  

**本篇源码示例:**
```
https://github.com/CainGao/SpringCloudAlibabaExample  
```
