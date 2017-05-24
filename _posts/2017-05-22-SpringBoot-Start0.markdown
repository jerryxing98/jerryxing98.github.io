---
layout:     keynote
title:      "SpringBoot-最佳实践-路线图"
subtitle:   "路线"
header-img: "img/post-bg-js-version.jpg"
navcolor:   "invert"
date:       2017-05-21
author:     "JerryMinds"
tags:
    - Java
    - SpringBoot
    - microservice
    - 云端
---


> 找到路就不嫌远

微服务图谱
----------
## 组织结构

### 全功能团队
### 去中心化
###　康威定律

## 理论基础
### 概念
#### 　独立性 - 独立测试，独立开发，独立部署
#### 　进程隔离 - 服务运行在独立进程中
#### 　多微合适 - 非代码函数,非重写时间,适合团队最重要
####　　轻量级 - 格式语言无关，协议跨平台


### 本质 
* 围绕业务组织团队
* 服务作为组件
* 演进式架构
* 基础设施自动化


### 优点 - 独立部署，按需伸缩，技术多样性，业务独立



### 缺点 - 测试成本高(自动化测试，契约测试) ，运维成本高（部署，监控，环境配置provisioning），依赖管理成本高（版本管理，服务依赖，服务治理）


### 与SOA区别

实现方式
部署方式
集成方式
服务粒度


## 常用模式

### 部署模式 - 单机多实例，单机单实例，容器多实例，容器单实例
### API网关  - 安全认证授权，响应合成，请求转发，协议转换
### 服务注册 - 自注册，第三方注册（consul,eureka）
### 服务配置 - 
### 服务发现 - 服务器端发现，库/工具


## 开发实践
* 服务说明文件 ： 请求/响应描述，运行环境，开发环境搭建，责任人，服务描述，监控告警，部署方式，测试策律
* 开发模板 ：springboot,springcloud,dropwizard  ,(持续集成：jenkins,bamboo)(部署脚本模板Chef,Shell,ansible,puppet)
* 服务结构 : 模型存储，业务模型，模型表示层，业务逻辑，集成网关


## 测试实践


单元测试
集成测试
组件测试 进程内，进程外
运维实践 告警（告警级别-Oncall,backup,owner,leader，告警方式，工具） 日志 ,监控(应用监控-（响应时间，健康性，关联ID,业务相关METRIC)，系统监控-CPU,MEMEORY,IO，工具-HOSTED(NAGIOS,ZABBIX),SAAS-NEWRELIC,ONEAPM)
部署实践 





