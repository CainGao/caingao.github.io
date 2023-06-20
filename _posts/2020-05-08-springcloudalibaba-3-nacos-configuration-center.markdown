---
layout: post
title: SpringCloudAlibaba(三)：使用Nacos作为配置中心
date: 2020-05-07 00:00:00 +0300
description: SpringCloudAlibaba(三)
img: springcloudalibaba/Spring.png # Add image post (optional)
tags: [SpringCloudAlibaba] # add tag
---
之前提到了这次从php技术栈迁移到java技术栈要解决的一个问题就是集中化配置管理。  

我们为什么会有配置中心的需求?  
1.  配置管理变更较为方便
2.  合理控制配置的权限内容  

### Nacos 控制台增加配置文件
1.  进入Nacos管理界面,在【配置管理】-【配置列表】功能页面点击右上角的 + 号。  
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_console.png)
2.  进入 新建配置 页面，填写要新增的配置内容  
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_add.png)  
**WARN**:Data ID的默认扩展名为**properties**,如果需要使用yaml格式则需要指明是 **.yaml**
3.  发布配置
配置完成后点击发布,即可在配置列表中看到刚才新增的配置

### 创建Nacos Config客户端
1.  新建项目,由于使用SpringCloudAlibaba直接引用相关依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
2.  创建应用主类并且实现一个HTTP接口
*   启动类
```java
@SpringBootApplication
public class NacosConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigApplication.class,args);
    }
}
```
*   Controller  
```java
@RefreshScope
@RestController
@RequestMapping("/config")
public class ConfigController {
    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @Value("${name}")
    private String name;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }

    @RequestMapping("/name")
    public String name(){
        return name;
    }
}
```
**@RefreshScope**在这里的作用就是让配置内容支持动态刷新，也就是当应用运行中，我们在Nacos控制台修改了配置之后这里也会动态的更新。

3.  项目bootstrap.properties配置服务名称与Nacos地址
```
# 配置中心url
spring.cloud.nacos.config.server-addr=localhost:8848
# 配置中心展现的服务名称
spring.application.name=nacos-config-example
#配置文件类型[TEXT,JSON,XML,YAML,HTML,Properties]
spring.cloud.nacos.config.file-extension=properties
# 配置分组,当前的业务基本选择为某些的GROUP,可以基于业务来划分不同的分组.
spring.cloud.nacos.config.group=DEFAULT_GROUP
# 默认选择的配置环境,当前把环境划分为4套, dev[开发环境],test[测试环境],pre[预发环境],prod[生产环境]
spring.profiles.active=dev
```
**WARN** 多环境配置中需要指定Nacos namespace的id，而不是指定namespace的名称 

4.  启动应用程序并进行验证  
*   启动应用  
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_start.png)  
*   发送请求测试配置是否生效  
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_test.png)  
*   修改配置进行进行动态刷新  
进入Nacos控制台修改配置信息,name修改为bigData 再次进行测试   
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_refresh.png)  
再次发送请求    
![]({{site.baseurl}}/assets/img/springcloudalibaba/3/nacos_web_config_test_refresh.png)

至此使用Nacos作为配置中心已经完全搞定,并且也实现了多环境的配置。多环境有几种方式实现，但是我依然习惯使用namespace的方式来实现。    
**本篇源码示例:**
```
https://github.com/CainGao/SpringCloudAlibabaExample  
```
    

