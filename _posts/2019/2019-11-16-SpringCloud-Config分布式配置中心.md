---
layout: post
title: 'SpringCloud Config分布式配置中心'
date: 2019-11-16
author: 白皓
cover: '/assets/img/little-boy.jpg'
tags:  SpringCloud  分布式    
---

##  SpringCloud Config简介
在分布式系统中，由于服务数量居多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。
在SpringCloud中，有分布式配置中心组件SpringCloud Config，它支持配置服务放在内存中(即本地)，也支持放在远程Git仓库中。在SpringCloud Config中有两个角色，
分别是Config Server和Config Client。

##  创建服务端

### 新建工程
新建SpringBoot工程`config-server`，`pom.xml`配置文件如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.springcloud.bh</groupId>
    <artifactId>config-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>config-server</name>
    <description>分布式配置中心-服务端</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```
主要是增加了`spring-cloud-config-server`的依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### Application
添加`@EnableConfigServer`注解，开启配置服务器功能
```java
package com.springcloud.bh.configserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

### application.yml
增加Config相关配置，并设置端口号:`8888`
```yaml
server:
  port: 8888   #默认8888

spring:
  application:
    name: config-server   #服务名称

  cloud:
    config:
      label: master   #配置仓库分支
      server:
        git:
          uri: https://github.com/yeshang5/springcloud-config-server.git   #仓库地址
          username:                #访问Git仓库的账号
          password:               #访问Git仓库的密码
          search-paths: respo   #配置仓库路径(存放配置文件的目录)

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/  #服务注册中心地址
```
相关配置说明如下:
*   `spring.cloud.config.label` : 配置仓库的分支
*   `spring.cloud.config.server.git.url` : 配置Git仓库地址
*   `spring.cloud.config.server.git.search-paths` : 配置仓库路径(存放配置文件的目录)
*   `spring.cloud.config.server.git.username` : 访问Git仓库的账号
*   `spring.cloud.config.server.git.password` : 访问Git仓库的密码

>   注意事项:
>*  如果使用GitLab作为仓库的话，`git.uri`需要在结尾加上`.git`，GitHub则不用

### 上传配置文件
在`springcloud-config-server`仓库的`Respo`文件夹中上传了`eureka-consumer-feign-dev.yaml`配置文件。

### 测试
打开浏览器访问:http://localhost:8888/eureka-consumer-feign/dev/master 即可访问到配置文件

![](https://s2.ax1x.com/2019/12/30/lM4X1f.png)
此时证明配置服务中心可以从远程仓库中获取配置文件

**附：HTTP请求地址和资源文件映射**
>*  http://ip:port/{application}/{profile}[/{label}]
>*  http://ip:port/{application}-{profile}.yml
>*  http://ip:port/{label}/{application}-{profile}.yml
>*  http://ip:port/{application}-{profile}.properties
>*  http://ip:port/{label}/{application}-{profile}.properties

##  创建客户端
此处直接改造`eureka-consumer-feign`项目，在`pom.xml`文件中添加依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### application.yml

将`eureka-consumer-feign`的配置文件放在Git仓库，本地application.yml只配置如下信息：
```yaml
spring:
  cloud:
    config:
      uri: http://localhost:8888   #配置中心服务端地址
      name: eureka-consumer-feign   #配置文件名
      label: master   #分支
      profile: dev    #配置文件环境标识
```
>   注意事项:
>*  配置服务器的默认端口为`8888`，如果修改了默认端口，则客户端项目不能在`application.yml`或`application.properties`中配置`sping.cloud.config.uri,
必须在`bootstrap.yml`或`bootstrap.properties`中配置，原因是`bootstrap`开头的配置文件会被优先加载和配置。

### 测试访问
启动`eureka-consumer-feign`项目，发现成功读取配置中心的配置并启动，端口号为`8765`
打开浏览器访问http://localhost:8765/hi?message=hi,Feign 网页显示
>   你的消息是：hi,Feign 端口号：8763

则成功启用SpringCloud Config实现分布式配置中心。

##  源码地址
**Github:** https://github.com/yeshang5/springcloud-example
 
##  配置文件仓库地址
**Github:** https://github.com/yeshang5/springcloud-config-server
