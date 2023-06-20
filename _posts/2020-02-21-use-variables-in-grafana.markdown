---
layout: post
title: 在Grafana中使用Variables
date: 2020-02-21 00:00:00 +0300
description: 使用Variables来更好的使用Grafana
img: grafana.png # Add image post (optional)
tags: [数据可视化, Grafana] # add tag
---
其实在公司内部使用Grafana完全是‘被逼的’。刚开始使用的时候完全是因为懒！也有考虑尽快的输出一些指标而不是走非常漫长的一个前、后、数据端的结伴开发。

ok，回归正题。

####    Grafana的Variables（变量）
Grafana提供了Variables来实现与面板交互的功能，并且能够实现动态的刷新面板。不需要进行使用SQL或者其他的方式实现修改面板。变量位于Dashboard的顶部，当变量发生改变时Dashboard会进行动态的变更。而所有相关依赖与变量的指标都会动态的发生改变。

![Variables]({{site.baseurl}}/assets/img/grafana_variables.png)   
Variables定义完成后我们能够直接通过Dashboard上的Variables进行动态的筛选Dashboard显示。例如常见的zabbix，即可直接通过定义的变量来变更服务器
实现动态的变更Dashboard中的数值。或以上图片通过变更应用ID来实现刷新Dashboard中的相关指标.

####    Variables定义
Grafana是一种面向数据监控的场景，定义变量是为了更加方便的实现我们的数据展现。例如常见的我们zabbix管理着10台服务器。
那么我们就可以通过定义服务器名称的方式进行动态的刷新Dashbaord指标。毕竟每台服务器的指标都是一致的。所以，变量就是在该Dashboard中表示为全局变量。

本次我们通过分析不同版本的在线人数与其他指标来进行变量的定义。

首先创建一个新的Dashboard，然后点击右上角的设置。进入Variables菜单。

当前是我已经建好的一个变量。

![Variables]({{site.baseurl}}/assets/img/grafana_variables_version.png)

可以点击new进行新建。
![Variables]({{site.baseurl}}/assets/img/grafana_variables_setting.png)

Type是变量的类型，当前版本一共由七种组成：Interval、Query、Datasource、Custom、Constant、Ad hoc filters和Text box。  
这些变量类型通常在一个Dashboard中可以组合进行使用。例如有一万台机器，我们在观测指标的时候通过前缀来筛选或者通过模糊来搜索的方式就可以通过Text box先输入几个关键词再通过mysql去检索来缩小范围等。

上图使用的是Query的方式检索ElasticSearch中的关键词来筛选版本信息。
```json
{"find": "terms", "field": "clientversion"}
```
![Variables]({{site.baseurl}}/assets/img/grafana_variables_preview.png)
当preview出现结果时即证明该变量设置成功，已经可以使用。

*   Refresh表示变量的刷新时间,有三种时间选择 
    *   Never:设置后用不刷新 
    *   On Dashboard Load:Dashboard重新加载的时候刷新一次。 
    *   On Time Range Change:跟随Dashboard的刷新时间进行刷新。Dashboard右上角。
*   Selection Options
    *    Multi-value: 多选
    *    Include All option:是否包含所有(会增加一个ALL的变量)
### 使用变量
当前我们已经通过设置了一个版本号的变量。那么就可以直接在查询的时候引用该变量来实现刷新Dashboard图表。  
首先新建一个图表。基于自己的业务选择相关数据源与图表类型等。  我们之前创建的variables叫做**clientVersion**，直接可以通过&clientVersion进行变量的引用。
例如:  
![Variables]({{site.baseurl}}/assets/img/grafana_variables_use.png)

这样,我们在选择不同的变量值时图片就会基于不同的变量值进行刷新了。我们也就能够基于版本的不同来获取指标的变化。

当然我们还可以通过设置更多的指标来实现更加精细的功能。例如我们当前版本来查看不同的地区指标信息。那么就可以再创建一个地区指标。

![Variables]({{site.baseurl}}/assets/img/grafana_variables_query.png)

如上,我们就可以基于版本号再查询该版本号下的地区信息来实现更细粒度的Dashboard图表变更。    
常用于分析某个版本在某个地区的使用指标情况，例如 不同区域的网络状况，请求失败情况等。

通过合理的配置指标，我们能够实现非常多的业务需求。我们为相同的指标分配了不同的维度。例如用户请求成功与失败可以分为不同的地区，不同的机房，不同的版本等。
那么相同的版本又可以分为不同的地区，不同的机房。相同的机房也可以分为不同的版本，不同的地区用户接入等等。

Variable在Dashboard查询、Title或Description上均可进行引用。来实现可视化数值、内容的动态变更。
