---
layout: post
title: MYSQL的奇怪问题:varchar与数值比较
date: 2020-05-22 00:00:00 +0300
description: MYSQL的奇怪问题:varchar与数值比较
img: mysql/mysql-log.png # Add image post (optional)
tags: [MYSQL,基础技能,奇怪问题] # add tag
---
我在工作中很少遇到所谓的‘奇怪的问题’。所以对于‘奇怪的问题’我还是很期盼的，可能很早的时候就被某些XX开发规范给限制住了，也就很少遇到这些所谓的奇怪的问题。所以严格来说 XXX开发规范 还是很靠谱的。  

好了来说具体场景，被同事叫去看一个奇怪的SQL。SQL语句很简单，大概就是查询某些字段有一些查询条件而已。 其中比较重要的一个条件就是 **where xx!=0**。说是很奇怪，为什么！=0就查询到的结果就是10条。但是！=1 查询出来的结果就是100条。  
下面我们简单的来几条数据看一下状况。  
### 状况复现
**所有数据**  
![]({{site.baseurl}}/assets/img/mysql/varchartoint/mysql_varchar_int_test_all.png)  
**!=0**  
![]({{site.baseurl}}/assets/img/mysql/varchartoint/mysql_varchar_int_test_notzero.png)    
当时就是直接拿到了这样的结果。

### 分析状况
当时看到这个问题之后我也很惊奇，不等于0 不应该把所有的数据都拿到么。为什么会出现这样的情况呢？  

简单考虑了一下，字段的类型为 varchar型，而查询条件给予的是个数值型，那么问题应该就是出现在这里。  

数据库在基于查询条件进行检索的时候会如何进行操作呢？   
答案就是转换成相同的类型。 

那么对于这次的问题是字段转换成int类型还是int转换成varchar类型呢？ 其实简单的看查询结果就知道了。如果查询条件‘0’转换成了varchar那么就应该获取到全部的数据。但是现在的状况是获取到的数据不够。那结论就是数据库把要查询的字段转换成了数值型。  
那么我们把app字段进行转换一下试试。  
![]({{site.baseurl}}/assets/img/mysql/varchartoint/mysql_varchar_int_test_cast_int.png)  
从结果上我们可以看到 app 转换之后的结果只有 0123asfj 转换成了123，其他都是0。 所以在查询条件为 ‘！=0’ 的时候就只能查询到一条结果。  

### 结论  
mysql在使用varchar字段查询条件是int类型的时候会把varchar型首先转换为int型进行查询。所以就会出现查询结果与预期不符的情况。另外如果字段类型是varchar型而查询条件使用int类型的话，查询是无法使用索引的，会进行全表的扫描。所以sql语句还是按照标准来写!

