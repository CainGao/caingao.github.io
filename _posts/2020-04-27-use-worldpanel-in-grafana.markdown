---
layout: post
title: Grafana的WorldMapPanel实现世界地图的分析展现
date: 2020-04-27 00:00:00 +0300
description: worldmap-panel插件的使用 
img: grafana-worldmap-panel-res.png # Add image post (optional)
tags: [基础技能] # add tag
---

**使用Grafana如果不画一个世界地图实在是有点可惜**

Grafana提供了一个worldmap-panel用来实现一个世界地图的数据可视化，常用来分析不同的地区不同的值。
例如最近大家都会关注的疫情地图。

![疫情地图]({{site.baseurl}}/assets/img/yiqingmap.png)

我们当前的需求其实很简单，想看一下我们当前的用户地区分布。如果仅仅是通过文字可能对于感官上来说不是很明显。  
所以，想通过数据可视化在一张世界地图上一目了然的看到。

### worldmap插件安装
Grafana的插件安装都比较简单，可以直接通过文档进行安装即可。  
https://grafana.com/grafana/plugins/grafana-worldmap-panel/installation

![安装插件]({{site.baseurl}}/assets/img/install-grafana-worldmap-panel.png)

安装完成后重启，即可在Visualization中看到该Panel。

### worldmap解决地图背景不显示的问题
可能很多人遇到了跟我一样的问题，插件安装了选择了该插件后无法显示地图背景。正常来说哪怕我们没有任何的数据也应该会有个世界地图展现出来。  
其实主要的问题就是网络的问题，worldmap-panel的访问地址需要进行修改。
1.  进入worldmap插件的安装目录备份出三个文件  
    1.1 grafana-worldmap-panel\src\worldmap.ts  
    1.2 grafana-worldmap-panel\dist\module.js   
    1.3 grafana-worldmap-panel\dist\module.js.map  
2.  将文件中的url进行修改.  
    2.1 https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png 修改为 http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png
    2.2 https://cartodb-basemaps-{s}.global.ssl.fastly.net/dark_all/{z}/{x}/{y}.png  修改为 http://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png

### worldmap-panel使用介绍
执行完成上面的修复之后worldmap已经能够正常显示地图的背景了，那么我们接下来就可以开始介绍worldmap-panel的使用了。  
worldmap-panel从插件的文档上就可以看到它支持的数据源包括 **Graphite**、**InfluxDB**、**OpenTSDB**、**Prometheus**、**MySQL**、**PostgreSQL**等数据源。
支持表数据与时间序列的数据形式。表数据的形式需要每条数据都带有geohash格式的经纬度坐标。

![Location Data]({{site.baseurl}}/assets/img/grafana-world-map-loc-data.png)

### 业务实现
需要实现一个用户所在地区的地图。数据的可以返回geohash。那么具体需要的数据就是 geohash，地区名称，人数。

![GeoHash Setting]({{site.baseurl}}/assets/img/grafana-worldmap-user-pv.png)

可以直接查看最终的结果

![worldmap]({{site.baseurl}}/assets/img/grafana-worldmap-panel-res.png)


至此图表展现完成！

