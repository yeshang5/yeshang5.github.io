---
layout: post
title: '使用jxls实现基于模板导出Excel'
date: 2018-10-22
author: 白皓
cover: 'https://s1.ax1x.com/2018/10/22/iDB1vq.jpg'
tags: java excel poi jxls 导出
---

##  前言

Java中实现excel根据模板导出数据的方法有很多，一般简单的可以通过操作POI进行。还可以使用一些工具很轻松的实现模板导出。这些工具现在还在维护，而且做得比较好的国内的有easyPOI，国外的就是这个JXLS了。

---
##  添加依赖

__Maven项目__：
```xml
    <dependency>
    <groupId>net.sf.jxls</groupId>
    <artifactId>jxls-core</artifactId>
    <version>1.0.6</version>
    </dependency>
```

__Gradle项目__
```java
    compile group: 'net.sf.jxls', name: 'jxls-core', version: '1.0.6'
```

##  forEach遍历导出

创建模板如图 [](https://s1.ax1x.com/2018/10/22/iDBNaF.png)
