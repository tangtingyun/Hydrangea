---
title: flutter调试
date: 2020-04-21 16:38:29
tags:
---

<!-- more -->

### flutter调试工具
<!-- more -->
- [Track widget rebuilds](https://flutter.dev/docs/development/tools/android-studio#show-performance-data)
  
  在as中打开 view -> toll windows -> flutter performance 可以看到界面widget在每一帧重新创建的次数 <br/>
  勾选 Track widget rebuilds选择框 可以看到界面所有元素的状态：

  每一列的介绍:  <br/>
  Widget: 页面中的组件 <br/>
  Location: 在源代码中的声明位置 <br/>
  Last Frame: `The exact count of the rebuilds for this frame displays` 在当前这一帧刷新的时候 该组件rebuild的次数 <br/>
  > 最右侧的图标状态: 如果构建次数过多 是黄色转圈圈 依次灰色转圈圈 灰色实心

  Current Screen: `how many times a widget was rebuilt since entering the current screen` 自从进入这个页面 该组件rebuild的次数