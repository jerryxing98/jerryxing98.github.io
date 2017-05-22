---
layout:     keynote
title:      "SpringSecurity-Api安全管理"
subtitle:   "Slides: SpringMVC-Api安全管理"
header-img: "img/post-bg-js-version.jpg"
navcolor:   "invert"
date:       2017-05-21
author:     "JerryMinds"
tags:
    - Java
    - SpringSecurity
    - 云端
---

> 通常情况下，把API直接暴露出去是风险很大的，不说别的，直接被机器攻击就喝一壶的。那么一般来说，对API要划分出一定的权限级别，然后做一个用户的鉴权，依据鉴权结果给予用户开放对应的API。目前，比较主流的方案有几种:


* 用户名和密码鉴权，使用Session保存用户鉴权结果(不适合移动时代)
* 使用OAuth进行鉴权（其实OAuth也是一种基于Token的鉴权，只是没有规定Token的生成方式）-（不做开放平台,太过复杂）
* 自行采用Token进行鉴权 （内部产品比较合适）


---
### 解决方案（JWT+SpringSecurity）

#### JWT介绍
#### SpringSecurity介绍
#### 集成及测试

