---
title: flutter学习记录
date: 2020-04-02 22:55:32
tags:
---

### 周末在medium上看了两篇关于flutter适配不同分辨率的文章 觉得挺好的
- [flutter适配不同分辨率](https://medium.com/flutter-community/flutter-effectively-scale-ui-according-to-different-screen-sizes-2cb7c115ea0a)
- [flutter适配](https://medium.com/flutter-community/flutter-sized-context-an-easier-way-to-access-mediaquery-size-gskinner-blog-f88147bb8aa7)
  
 看文章的时候遇到一个dart的知识点(学习flutter坑定绕不过dart的) 通过dart的扩展函数 把用媒体查询的到的单位宽度和单位高度扩展到context对象上 从而方便使用 [extension](https://dart.dev/guides/language/extension-methods)
 <!-- more -->