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

### 关系的种类
关联，泛化，包含，扩展
![img](/img/in-post/post-sofeware/software-usecase-relation.png)


1.关联(Association)：表示参与者与用例之间的通信，任何一方都可发送或接受消息。
2.泛化(Inheritance)：就是通常理解的继承关系，子用例和父用例相似，但表现出更特别的行为；子用例将继承父用例的所有结构、行为和关系。子用例可以使用父用例的一段行为，也可以重载它。父用例通常是抽象的
3.包含(Include):包含关系用来把一个较复杂用例所表示的功能分解成较小的步骤。
4.扩展(Extend):扩展关系是指用例功能的延伸，相当于为基础用例提供一个附加功能。


