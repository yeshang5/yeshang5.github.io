---
layout: post
title: 'OpenLayers  实现动画方式移动和缩放(OpenLayers View animate)'
date: 2018-12-25
author: 白皓
cover: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1545734973331&di=a7b83cde099882bb013caa650718258e&imgtype=0&src=http%3A%2F%2Fboundlessgeo.com%2Fwp-content%2Fuploads%2F2016%2F11%2FOL3-300x300.png'
tags: openlayers 地图 GIS
---


Openlayers 中文教程可参考 http://anzhihun.coding.me/ol3-primer/

项目中要实现把OpenLayers的View移动到某点并放大到指定级别，可使用以下方法

```py
  view.animate({zoom: 10}, {center: [0, 0]});
```

>  经过实际使用，发现传递给animate方法的参数，编写的先后顺序会影响动画效果，下面记录一下！

```py
  //注意此处坐标需要将GPS坐标转换为国家测绘局坐标
  var point = ol.proj.transform([lng, lat], 'EPSG:4326', 'EPSG:900913');   

  // 动画移动
  // 先移动再放大
  map.getView().animate({center: point}, {zoom: 18}) 

  // 先放大再移动
  map.getView().animate({zoom: 18}, {center: point}) 

  // 同时移动和放大
  map.getView().animate({center: point, zoom: 18}) 
```

故根据以上可以实现一些列的动画组合，如实现先缩小定位置某点再放大，
```py
  map.getView().animate(
    {
      center: point, 
      duration: 1000,   //地图飞行时间，毫秒 
      zoom: 8，
    {
      zoom:16,
    }
  ) 
```

> 先移动再放大

![](https://g1.ax1x.com/2018/12/25/Fc612Q.gif)

> 先放大再移动
![](https://s1.ax1x.com/2018/12/25/Fc6QPS.gif)

> 同时移动和放大
![](https://s1.ax1x.com/2018/12/25/Fc6l8g.gif)