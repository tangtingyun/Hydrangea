---
title: Android启动优化
date: 2021-03-06 15:38:10
tags:
---
Android启动优化记录
<!-- more -->

## 启动分类

- 冷启动
应用从头开始启动 进程不存在
加载并启动App  －> 启动后立即为该App显示一个空白启动窗口（黑白屏问题）
-> 创建App进程  －> 启动主Activity - > 加载布局 绘制
- 热启动
在热启动中 系统的所用工作就是将Activity带到前台 只要应用的Activity仍驻留在内存中 应用就不必重复执行对象初始化 布局加载和绘制
比如按下Home键
- 温启动 
温启动包含了在冷启动期间发生的部分操作 同时 它的开销要比热启动高
>例如：
1.用户back back退出了应用又重启。进程可能未销毁，继续运行 但应用需要执行onCreate从头开始重新创建Activity
2.系统将应用从内存中释放，然后用户又重启它，进程和Activity需要重建 但
传递到onCreate的已保存的实例savedInstanceState对于完成此任务有一定帮助。

## 启动耗时统计

1. 系统日志统计
   过滤 display关键词 选择No Filters 即可看到当前页面启动时间
   或者过滤 ActivityManager 可以看到更到AMS日志信息

2. adb命令统计
   adb shell am start -S -W packagename/activityname

3. 启动时 通过CPU Profile观察调用栈 信息更详细
4. 手动打点

## 启动黑白屏

- 通过设置主题的方式 在视觉效果上美化初始化效果 这么做 只是提高启动的用户体验，并不能做到真正的加快启动速度
- windowDisablePreview 设置系统的取消预览
- windowIsTranslucent 设置背景透明
- windowBackground 设置window背景
- windowNoTitle  
- windowFullscreen

## 启动优化方案

- 合理的使用异步初始化 延迟初始化 懒加载机制
- 启动过程避免耗时操作 如数据库I/O 操作不要放在主线程执行
- 类加载优化：提前异步执行类加载
- 合理使用IdleHandler进行延迟初始化
- 简化布局
- 拆分资源 
- 在debug模式下 开启StrictMode 它可以检测除我们可能无意中做的事情 并提醒给我们
- 优化是一条持续的道路.....

##  总结分类

- 启动优化 一定是减少时间

- 业务流程优化 视觉欺骗
  预览窗口优化 业务流程优化 延迟加载 动画效果等

- 代码优化 减少加载时间
  UI优化 内存优化 图片优化....