---
layout: post
title: '使用openLayer实现热力图'
date: 2019-3-22
author: 白皓
cover: 'https://s2.ax1x.com/2019/09/02/nCMlNj.jpg'
tags: markdown  emoji
category: markdown
---

```javascript
var self = this;
        this.map = map = self.map || window.map;
        var pointList=new Array();
        // ctx+"/basedata/hiddenPointInfo/data"
        // "https://easy-mock.com/mock/5bd6944ee4a7377be953147c/example/getList"  隐患点数据Mock，用于测试
        app.post_async(ctx+"/basedata/hiddenPointInfo/data",null,false,function (data) {
            if (data.rows.length>0)
            {
                pointList=data.rows;
            }
        });

        var features = new Array(pointList.length);
        for (var i = 0; i < pointList.length; i++) {
            var coordinate = ol.proj.gps2gcj(pointList[i].lng,pointList[i].lat);
            features[i] = new ol.Feature({
                geometry: new ol.geom.Point(coordinate),
                weight:0.6
            });
        }

        var heatMapLayer=new ol.layer.Heatmap({
            title: "隐患点密度",
            weight: weightFunction,//设置权重,值在0-1之间
            gradient: self.gradient,
            blur: 15,//默认15
            radius: 25 || 8,//默认8,
            renderModed:'image',//图层渲染方式，image和vector分别为栅格和矢量，第一个渲染速度快；后者慢，ol5新增加的属性，对于大量数据渲染有利
            source: new ol.source.Vector({//热力图数据来源
                features: features
            }),
        })

        function weightFunction(feature) {
            var weight = feature.get('weight');
            weight = parseFloat(weight);
            //weight = parseFloat(weight) / 10;
            return weight;
        }

        this.map.addLayer(heatMapLayer);
```