---
layout:     post
title:      "大型互联网应用中的一些常见数据指标"
subtitle:   ""
date:       2015-12-10 14:00:00 +0800
author:     "颜俊标"
header-img: "img/post-bg-2015.jpg"
tags:
    - 数据
---
## 数据库
### 关系型数据库
响应时间一般在10ms以内

几亿条数据，1万TPS，响应时间也要在10毫秒以内，几乎任何一款关系型数据库都无法做到

#### MySQL
TPS： 1500
单表数据条数一般应该控制在2000万以内

MySQL 5.7.4 

### 缓存存储
响应时间要求在1～2ms内返回数据

#### memcached
单节点 15万 TPS， 百倍的MySQL TPS

## 网络
大型IDC内部，一个TCP回环： 0.2～0.5ms

