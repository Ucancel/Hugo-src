---
title: Design Principle
author: Jiaqi Gao
description: 
date: 2023-06-03T00:31:36+08:00
lastmod: 2023-06-03T00:31:36+08:00
slug: design-principle
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories: 
  - DesignPatterns
tags:
series:
aliases:
---

### I. 开闭原则｜Open-Closed Principle｜OCP
> 对扩展开放，对修改关闭。

### II. 依赖倒置原则｜Dependence Inversion Principle｜DIP
> 高层模块不应该依赖底层模块，二者都应该依赖其抽象。
> 抽象不应该依赖细节，细节应该依赖抽象。

### III. 单一职责原则｜Simple Responsibility Pinciple｜SRP
> 不要存在多于一个导致类变更的原因。

### IV. 接口隔离原则｜Interface Segregation Principle｜ISP
> 使用多个专用接口，不使用单一的总接口。

### V. 最少知道原则｜Least Knowledge Principle｜LKP
> 一个对象应该对其他对象保持最少的了解。

### VI. 里氏替换原则｜Liskov Substitution Principle｜LSP
> 若对于任意的类型为`T1`的对象`O1`，都有类型为`T2`的对象`O2`，使得以`T1`定义的所有程序`P`在所有的对象`O1`都替换成`O2`时，程序`P`的行为没有发生变化，则类型`T2`是类型`T1`的子类型。

**引申含义：**
> 子类可以扩展父类的功能，但不能改变父类原有的功能。

### VII. 合成复用原则｜Composite / Aggregate Reuse Principle｜CARP
> 尽量使用对象组合/聚合而不是继承关系达到软件复用的目的。
> **白箱复用：** 指继承，把所有的实现细节暴露给子类。
> **黑箱复用：** 指组合/聚合，无法获取类以外的实现细节。

### 总结
**设计原则是设计模式的基础，不要求代码完全遵循设计原则，适当的使用设计原则，不要刻意追求完美。**