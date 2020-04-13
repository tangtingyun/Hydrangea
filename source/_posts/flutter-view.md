---
title: flutter-view
date: 2020-04-13 20:53:20
tags: flutter
---

flutter中构建UI有三个概念
<!-- more -->
[官方视频](https://www.youtube.com/watch?v=996ZgFRENMs)

- Widget tree <br/>
    `A widget is an immutable description of part of user interface`<br/>
    `describes the configuration for an Element` <br/>
     widget是不可变 是你UI的一个简单抽象描述 
- Element tree <br/>
    `an instantiation of a widget at a particular location in the tree` <br/>
     element是widget树中某个widget的具体实例
- RenderObject tree <br/>
    `handles size layout, and painting.` <br/>
    底层劳动者 真正干活的 测量 绘制 布局

为什么要有三棵树呢? <br/>
  重(复)用 提高性能<br/>
从上到下 一个比一个重,构建代价高.<br/>
因此当用户界面发生变化时候 首先比较widget
```
  static bool canUpdate(Widget oldWidget, Widget newWidget){
      return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key // 如果你构建的widget没传key 就不会比较key
  }
```
1.如果可以重用 链接旧的element元素更新旧的RenderObject的属性重新渲染<br/>
- *如果你有个stateful列表并且有增删改查的操作 最好给widget绑定唯一key 防止因为重用导致数据不能及时更新*
- *如果是stateless列表 就不用担心 因为每次都会创建新实例*

2.如果不可以重用 创建新的element 新的RenderObject渲染