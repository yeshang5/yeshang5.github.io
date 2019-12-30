---
layout: post
title: 'SpringCloud-SpringBoot Admin实现服务监控'
date: 2019-11-17
author: 白皓
cover: '/assets/img/little-boy.jpg'
tags: SpringCloud  分布式  
---

##  创建服务端
新建SpringBoot工程`springcloud-admin`,`pom.xml`文件配置如下:
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
    <artifactId>apringcloud-admin</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>apringcloud-admin</name>
    <description>服务监控</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-boot-admin.version>2.2.0</spring-boot-admin.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--<dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
        </dependency>-->
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
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
            <dependency>
                <groupId>de.codecentric</groupId>
                <artifactId>spring-boot-admin-dependencies</artifactId>
                <version>${spring-boot-admin.version}</version>
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

主要增加了`spring-boot-admin-starter-server`的依赖
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
```

### Application
通过`@EnableAdminServer`注解开启Admin服务监控功能。
```java
package com.springcloud.bh;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableAdminServer
@EnableEurekaClient
public class ApringcloudAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApringcloudAdminApplication.class, args);
    }
}
```

### appliction.yml
设置端口号为:`8084`
```yaml
server:
  port: 8084

spring:
  application:
    name: springcloud-admin-server

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/


```

##  创建客户端
在之前的项目[`springcloud-example`](https://github.com/yeshang5/springcloud-example)的每个Module项目中添加pom依赖
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-client</artifactId>
    <version>2.1.0</version>
</dependency>
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
    <version>1.6.2</version>
</dependency>
```

在`application.yml`中添加以下配置
```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:8084  #springboot-admin服务监控server地址
```

## 测试访问

启动所有项目后访问http://localhost:8084 即可看到服务监控页面

![](https://s2.ax1x.com/2019/12/30/lQwPjP.png)

##  源码地址
**Github:** https://github.com/yeshang5/springcloud-example