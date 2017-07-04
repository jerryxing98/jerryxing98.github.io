---
layout:     post
title:      "软件工程系列-质量控制"
subtitle:   ""
date:       2017-06-02 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 云端
    - 软件工程
    - 质量控制
---



#### 依赖管理
用maven的依赖插件的分析结果来帮助你识别项目中的依赖管理问题

maven依赖插件（Maven Dependency Plugin）的analyze 目标可以识别一个项目中关于依赖的两类问题：
* 直接就使用但未声明的依赖；（项目可以编译因为它也得到了所需的依赖）



有如下的假定:
```
A项目使用了来自B和C项目的类；
A只声明了一个B项目的依赖；
B项目也使用了C项目的类；
B项目声明了以一个C项目的依赖。
```
这种情况存在一个潜在的问题，因为A依赖B，B依赖C，A项目只有在B项目移除了对C项目的引用后才能编译。如果是SNAPSHOT版本的话问题将会更加严重。

当对A项目用mvn dependency:analyze进行分析时，maven将会有如下输出：
```
[WARNING] Used undeclared dependencies found:
[WARNING] com.avaya.ace.aaft:foundation_services_client_api:jar:6.2.0-SNAPSHOT:compile
```
A项目直接声明一个对C项目的引用就会解决这个问题。



* 声明了但没有使用的依赖
如果一个项目声明了一个依赖，但没有使用它，mvn dependency:analyze会有如下输出：
```
[WARNING] Unused declared dependencies found:
[WARNING] com.gigaspaces:gs-openspaces:jar:7.1.1-b4534:compile
```
在POM文件移除这个依赖声明就可以解决这个问题。
未使用的依赖一般是开发者没有搞清楚maven dependenceManagement的原理造成的。


* 不要写代码复制解压文件
使用maven标准的插件来替代。

* 良好的mvn依赖代码规范和分层
分为这几个层的依赖 
1.Common 通用的依赖 Log,Json,工具类
2.Web 类的依赖 前端模块依赖，API工具等 
3.Model层的依赖 Lombok之类的依赖
4.DAO层的依赖，同数据访问相关的依赖，数据库依赖包
5.Service层依赖 ，中间件的依赖，Spring等
6.业务第三包


#### CodeStyle 

