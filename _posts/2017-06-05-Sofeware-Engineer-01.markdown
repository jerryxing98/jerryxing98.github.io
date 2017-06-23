---
layout:     post
title:      "软件工程系列-UML建模-用例图"
subtitle:   ""
date:       2017-06-02 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 云端
    - 软件工程
    - 用例图
---

> 如何描述一个系统?如何帮助开发团队理解系统的功能需求

### 用例图所包含的元素
* 参入者
* 用例
* 子系统
* 关系

### 用例关系的种类
关联，泛化，包含，扩展
![img](/img/in-post/post-sofeware/software-usecase-relation.png)


1.关联(Association)：表示参与者与用例之间的通信，任何一方都可发送或接受消息。
2.泛化(Inheritance)：就是通常理解的继承关系，子用例和父用例相似，但表现出更特别的行为；子用例将继承父用例的所有结构、行为和关系。子用例可以使用父用例的一段行为，也可以重载它。父用例通常是抽象的
3.包含(Include):包含关系用来把一个较复杂用例所表示的功能分解成较小的步骤。
4.扩展(Extend):扩展关系是指用例功能的延伸，相当于为基础用例提供一个附加功能。


### 部署图
部署图描述的是系统运行时的结构，展示了硬件的配置及其软件如何部署到网络结构中。一个系统模型只有一个部署图，部署图通常用来帮助理解分布式系统。

* 结点
  结点是存在与运行时的代表计算机资源的物理元素，可以是硬件也可以是运行其上的软件系统，比如linux主机，防火墙等。用三维盒装表示
* 结点实例(Node Instance)
  Node Instance : node
* 结点类型（Node Stereotypes）
* 物件（Artifact） 
  物件是软件开发过程中的产物，包括过程模型（比如用例图、设计图等等）、源代码、可执行程序、设计文档、测试报告、需求原型、用户手册等等。
* 连接（Association）
* 结点容器（Node as Container）
  一个结点可以包括其他的结点，比如组件或者物件，则称此结点为结点容器（Node as Container）。如下图所示，结点（Node）包容了物件（Artifact）。


### 组件图
https://www.ibm.com/developerworks/cn/rational/rationaledge/content/feb05/bell/bell.html
