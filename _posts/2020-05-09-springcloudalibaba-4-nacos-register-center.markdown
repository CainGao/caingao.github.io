---
layout: post
title: SpringCloudAlibaba(四)：使用Nacos作为注册中心
date: 2020-05-09 00:00:00 +0300
description: SpringCloudAlibaba(四)
img: mac.jpg # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
前一篇已经写到了利用Nacos作为配置中心来构建一个项目，毕竟需求有很大一部分的比重就是配置中心。所以就先利用Nacos构建了基于配置中心的项目。  
Nacos作为注册中心是更加常用的。下面创建项目注册到Nacos中。

### 服务提供者
1.  创建一个项目作为服务的提供者 **nacos-discovery-http-provider**
2.  maven引入相关依赖
    ```xml
    <dependencies>
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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    ```
3.  创建相关服务类  
    **NacosHttpServerApplication.java**
    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class NacosHttpServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(NacosHttpServerApplication.class, args);
        }
    }
    ```  
    **UserController.java**
    ```java
    @RestController
    public class UserController {
        //提供相关服务
        @RequestMapping(value = "selectOne",method = RequestMethod.GET)
        public String selectOne(@RequestParam String id){
            return "Hello:"+id;
        }
    }
    ```
    *@EnableDiscoveryClient*表示开启SpringCloud服务注册注册发现功能。主要是由**pom.xml**中引入的 *spring-cloud-starter-alibaba-nacos-discovery* 模块实现.
4.  配置文件
    ```properties
    server.port=8081
    # 注册中心
    spring.cloud.nacos.discovery.server-addr=localhost:8848
    # 配置中心展现的服务名称
    spring.application.name=nacos-discovery-http-provider
    ```
5.  启动应用程序&查看Nacos控制台
    启动应用程序，当控制台输出以下内容时表示服务注册&启动成功
    ```
    2020-05-19 21:15:22.471  INFO 4568 --- [           main] c.a.c.n.registry.NacosServiceRegistry    : nacos registry, DEFAULT_GROUP nacos-discovery-http-provider 192.168.1.16:8081 register finished
    ```
    登陆Nacos控制台服务列表即可看到如下内容。  
    ![]({{site.baseurl}}/assets/img/springcloudalibaba/4/nacos_discovery_http_provider.png)

### 服务消费者
1.  创建服务消费项目 **NacosHttpConsumerApplication**
2.  编辑pom.xml文件,内容与provider的pom文件内容一致
3.  创建相关类
    **NacosHttpConsumerApplication.java**
    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableFeignClients("top.anydata.products.web.example.nacos_discovery_http_consumer.service")
    public class NacosHttpConsumerApplication {
        public static void main(String[] args) {
            SpringApplication.run(NacosHttpConsumerApplication.class,args);
        }
    }
    ```
    **IServerUserServiceFeign.class**
    ```java
    @FeignClient(value = "nacos-discovery-http-provider")
    public interface IServerUserServiceFeign {
    
        @RequestMapping(value = "selectOne",method = RequestMethod.GET)
        String selectOne(@RequestParam("id")String id);
    
    }
    ```
    **ClientUserController.java**
    ```java
    @RestController
    public class ClientUserController {
        @Autowired
        IServerUserServiceFeign serverUserServiceFeign;
    
        @RequestMapping(value = "selectOne",method = RequestMethod.GET)
        public String selectOne(String id){
    
            return serverUserServiceFeign.selectOne(id);
        }
    
    }
    ```
4.  配置文件配置服务名称与Nacos相关，实现发现Nacos中的服务
    ```properties
    spring.cloud.nacos.discovery.server-addr=localhost:8848
    # 配置中心展现的服务名称
    spring.application.name=nacos-discovery-http-consumer
    # 默认选择的配置环境,当前把环境划分为4套, dev[开发环境],test[测试环境],pre[预发环境],prod[生产环境]
    spring.profiles.active=dev
    management.endpoints.web.exposure.include='*'
    ```
5.  启动服务&测试调用  
    启动consumer服务，调用rest相关接口进行测试。  
    ![]({{site.baseurl}}/assets/img/springcloudalibaba/4/nacos_discovery_http_provider.png)

**自此即实现了使用Nacos作为注册中心来进行服务的管理**

**本篇源码示例:**
```
https://github.com/CainGao/SpringCloudAlibabaExample  
```
