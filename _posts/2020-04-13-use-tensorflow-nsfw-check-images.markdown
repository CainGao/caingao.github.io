---
layout: post
title: 深夜,使用NSFW测一下鉴黄
date: 2020-04-13 00:00:00 +0300
description: 使用Tensorflow-NSFW测试一下图片鉴黄
img: cangjingkong.jpeg # Add image post (optional)
tags: [Tensorflow, NSFW] # add tag
---
![导师]({{site.baseurl}}/assets/img/cangjingkong.jpeg)  
前几天公司在讨论鉴黄的问题，对接了一些厂家提供的鉴黄服务。由于公司本身就是做音视频领域相关的，鉴黄的需求量较大。
同时秉着好好学习，天天向上的精神。想自己试一下鉴黄相关的东西。 刚好同事也提出了一个开源库。顺便的了解了一下。  
当然本人纯粹是对于知识的渴求与好奇才尝试了一下，对于什么吉泽明步、小泽玛利亚、波多野结衣、饭岛爱、苍井空、武藤兰、麻生希...等等是绝对不认识的。都是为了学习...嗯~为了学习!  

#### NSFW
开始之前了解首先要先了解一下**NSFW**，**NSFW**是（*Not Safe For Work*）的意思,不适宜工作场所。(嗯~也就是说不适合工作的时候观看...) 是由yahoo开源的一套鉴黄的模型。  
https://github.com/yahoo/open_nsfw  
使用Caffe模型训练尔来，主要是针对恐怖，血腥，色情等图片进行鉴别。

####    快速开始
项目的markdown中说明了使用方式，非常简单的Docker一键启动。大家可以去github直接查看使用方式。    
```
docker build -t caffe:cpu https://raw.githubusercontent.com/BVLC/caffe/master/docker/cpu/Dockerfile
```
嗯...好吧,无法访问。给出了一个·假地址·。那么只能选择其他的方案了。  
自行安装失败...Caffe环境没有安装成功,国外的环境基本都是Ubuntu,而国内的都是CentOS。尝试N次。放弃！    
后来发现Tensorflow-nsfw的版本。

https://github.com/mdietrichstein/tensorflow-open_nsfw  

当前支持python3.6 与 tensorflow 1.12。已经通过相关测试，其他版本请慎重测试。尽量选择相同版本。  
下载完成环境正常的话直接可以通过
```python
python classify_nsfw.py -m data/open_nsfw-weights.npy test.jpg 
```
进行测试。 **注意:当前仅支持jpeg格式图片**
```json
Results for 'test.jpg'
	SFW score:	0.9355766177177429
	NSFW score:	0.06442338228225708
```
NSFW score 就是识别出不适合与工作场景的分值。总分为10分，分值越高，表示该图片越不适合与工作场景。  

####    测试
说实话图片还真是难找，特别是对我这种完全就找不到哪里的图片能让分值变成8分以上~
利用搜索引擎找了几张图片，个人觉得应该要8分以上了。但是结果...

苍井空  
![导师]({{site.baseurl}}/assets/img/cangjingkong.jpeg) 
t1  
![t1]({{site.baseurl}}/assets/img/t1.jpeg)
t2  
![t2]({{site.baseurl}}/assets/img/t2.jpeg)
t3 
![t3]({{site.baseurl}}/assets/img/t3.jpeg)

结果分值  
![score]({{site.baseurl}}/assets/img/nsfw_score.png)
但是计算结果仍是差强人意！

####    结果
基于最终的结果，可能我本人的知识有限，很难获取到能够达到8分以上的图片。只能期望大家俩给我一些灵感或者..嗯~~~你懂得!让我真实的来尝试一下 nsfw的鉴别效果!  大家可以给我留言告诉我从哪里可以让分值变成8以上。感谢大家！
