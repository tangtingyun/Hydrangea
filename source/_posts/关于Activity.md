---
title: 关于Activity
date: 2021-04-05 22:21:48
tags: Android
---

Activity的零碎知识点
<!-- more -->

#### 如何跨App 启动Activity

- 共享uid的App
- 使用exported = true 
- 使用隐式的intentFilter

防护措施
- 为允许外部启动的Activity加权限 permission/uses-permission
- 拒绝服务漏洞  启动intent时候 传入一个未序列化的类 
  - try catch

#### 任意位置为当前Activity添加View

- 获取当前Activity  通过 ActivityLifecycleCallbacks 回调 弱引用 activity
- 添加view
  - 获取 decorView 

#### onActivityResult 为什么不设计成回调

匿名内部类引用了外部实例 例如在回调里面使用了外部TextView
但是当回来的时候 出现了Activity销毁和恢复机制 导致引用失效


