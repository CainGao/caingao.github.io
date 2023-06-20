---
layout: post
title: 在Grafana中使用Variables(Global Variables)
date: 2020-02-21 00:00:00 +0300
description: 使用Variables来更好的使用Grafana(2)
img: grafana.png # Add image post (optional)
tags: [数据可视化, Grafana] # add tag
---
在前一篇<在Grafana中使用Variables>中已经提到过了Variables的使用，Grafana提供的Variables方式能够自由的切换数据进行展现。但是之前提到的Variables都是基于我们自身的数据源或者是我们的输入。   Grafana同时也支持一些Global的Variables提供给我们进行使用。

### 时间范围变量
在Grafana中筛选时间的范围可以直接在右上角进行筛选，但是可能我们实现某个需求的时候需要用到当前页的开始与结束时间。那么就可以用到时间范围变量。  
**$__from**, **$__to** 用于获取到当前Dashboard的时间范围。  
例如在钻取分析时需要获取到当前的时间与跳转页的时间对应。则可以直接在连接中增加 ++from=$__from&to=$__to++ 来实现时间统一。  
![Variables]({{site.baseurl}}/assets/img/grafana-variable-global_1.png)

或直接基于图表展现则可以直接在图表中使用 $__from $__to 实现展现.  
![Variables]({{site.baseurl}}/assets/img/grafana-variable-global_2.png)

### 间隔变量
在进行图表展现时,我们选择每分钟的最大在线人数。展现一个小时有60个点。但是如果要展现3天的话就需要4320个点。7天的话可能更多。  
通过较大的时间间隔来进行分组能够提高查询效率，如果点数多与图表上的话图表上也无法展示。例如，7天的时候我们可能希望能够按照小时进行分组。那么我们就可以使用 **$__interval**与**$__interval_ms**变量。   
$__interval是带有时间单位的，例如2m，1h，1d等这种。  
![Variables]({{site.baseurl}}/assets/img/grafana-variable-global_3.png)

### timeFilter
timeFilter实现时间表达式的生成，例如我们在直接使用SQL的时候就可以看到该变量与生成的结果。  
![Variables]({{site.baseurl}}/assets/img/grafana-variable-global_4.png)

### 其他
其他也存在一些变量,例如 $__name $__range $__dashboard $__org 等.某些参数可能仅针对某些数据源提供相关的支持

### 使用
Global Variables大多数应用与下钻分析的url上. 可以直接针对于下钻的参数进行调整。 常见的例如 **from** **to** 与自定义的 **var-id** 等。
