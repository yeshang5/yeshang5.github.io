---
layout: post
title: 'SpringCloud-使用Hystrix Dashboard熔断器仪表盘监控'
date: 2019-11-15
author: 白皓
cover: '/assets/img/little-boy.jpg'
tags:  SpringCloud  分布式 
---

##  添加依赖

打开之前的[`eureka-consumer-feign`](http://baihao520.com/2019/11/14/服务消费者(Feign))项目，在`pom.xml`中添加依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

## Application

在Application中添加`@EnableHystrixDashboard`注解，开启熔断器监控仪表盘
```java
package com.springcloud.bh;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients          /*启用Feign*/
@EnableDiscoveryClient      /*开启服务发现*/
@SpringBootApplication
@EnableHystrixDashboard    /*开启熔断器监控仪表盘*/
public class EurekaConsumerFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerFeignApplication.class, args);
    }

}
```

##  创建hystrix.stream的servlet配置

SpringBoot 2.x的版本中开启Hystrix Dashboard与SpringBoot 1.x的方式略有不同，需要增加一个`HystrixMetricsStreamServlet`的配置。

在`config`包里新建`HystrixDashboardConfiguration`配置类，代码如下：
```java
package com.springcloud.bh.config;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HystrixDashboardConfiguration {

    @Bean
    public ServletRegistrationBean getServlet()
    {
        HystrixMetricsStreamServlet streamServlet=new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean=new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.setName("HystrixMetricsStreamServlet");
        registrationBean.addUrlMappings("/hystrix.stream");
        return registrationBean;
    }
}
```

##  测试Hystrix Dashboard

浏览器访问http://localhost:8765/hystrix
![](https://s2.ax1x.com/2019/12/26/lAIcm6.png)

路径填写配置好的servlet路径`http://localhost:8765/hystrix.stream`,Delay表示监测延时，Title为熔断器标题，可自己取名。

点击Monitor Stream，进入熔断器监控详情页面，此时我们关闭`eureka-client`服务提供者，访问http://localhost:8765/hi?message=hi,Feign

此时服务会被熔断，而监控仪表盘上会成功显示服务熔断详情。
![](https://s2.ax1x.com/2019/12/26/lAIOAS.png)

>   注:Ribbon项目的改造步骤同Feign一样，因此此处只实现了Feign项目的熔断器仪表盘监控。

##  源码地址
**Github:** https://github.com/yeshang5/springcloud-example