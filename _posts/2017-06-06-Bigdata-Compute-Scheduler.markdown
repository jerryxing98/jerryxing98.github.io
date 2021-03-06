---
layout:     post
title:      "大数据系列-触发式计算"
subtitle:   ""
date:       2017-06-06 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 大数据
    - 云端
    - DataCompute
---

### 什么是触发式计算
先回顾下目前比较主流的几种大数据计算：
* 离线计算：对应的是批处理，响应时间是比较慢的，一般都在分钟级 甚至 小时级以上，典型的有：hadoop,spark 等
* 实时计算：对应的是即时查询，关注rt, 要求在秒级 甚至 毫秒级返回，典型的有：impala,pestro等
* 流式计算：对应的是pipeline式的数据处理，关注处理时效性。典型的有：storm,spark streaming等

什么是触发式计算？首先分两步来看这个定义
* 触发式：底层的计算逻辑是由用户在使用时决定，而不是事先预定义好，即底层的计算逻辑完全由用户在使用时选择的条件决定，而且由用户触发开始进行计算，计算时间可以很长，也可以很短，取决于底层实现机制。比较容易混淆的一个定义是：交互式，计算逻辑也是由用户在使用时决定，不同的是：交互式一般都要求结果即时返回，而触发式只是让用户根据自己需求进行启动，结果不一定即时返回.

* 计算：这里指的是大数据计算，可以是上面提到的离线计算，实时计算 或者 流式计算。

* 总的来说，触发式计算就是：由用户在使用时决定底层的计算逻辑，这个逻辑的实现可能是基于：离线计算(odps sql/mr, odps xlib,spark), 实时计算(garuda) ,或者混合离线和实时计算等。 当然这个并非业界通用的定义，是根据这两年在业务中摸索总结出来。触发式计算并非一种新的计算框架，只是将目前主流的大数据计算进行灵活组合封装，来满足上层的业务场景

### 应用场景
* 产品端的通过算法返回计算


### 实现方案
触发式大数据计算在实现上核心要解两个问题：

* 上层业务逻辑的灵活表达
一般的业务逻辑都可以理解为一个工作流，即一个DAG图。即业务逻辑可以进行分解，用一个DAG图进行表达。举个例子：上面的id转换服务可以分解为 1/ 将输入表中的数据上传 2/ 对数据进行转换 3/ 将结果数据存放在输出表中 4/ 将输出表输出给用户. 如果业务逻辑就是一个dag图，那要解的就是如何灵活生成一个类似dag的工作流.

* 底层大数据系统的对接
一般来说，每个大数据系统都有对应的api进行交互， 比如hive提供可以运行sql或者mr的api, garuda提供jdbc的方式进行查询。 上层业务逻辑生成一个类似dag的工作流，那么工作流中每个节点的实现就是通过这些api来完成。


实现方案1：裸实现
使用java来实现业务逻辑对应的流程，使用底层大数据系统的api进行交互。但这个方案有两个缺点：

* 需要使用java代码来硬编码业务逻辑，不够灵活和扩展
* 需要维护跟底层数据系统api交互

这种方式开发效率不高.


