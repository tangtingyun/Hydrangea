---
title: Opencv集成
tags:  opencv
---

[参考链接](https://juejin.im/post/5d46d1e8e51d45620541034c#heading-11)
[参考链接](https://juejin.im/post/5d984028518825095879e45d#heading-15)
[opecv集成](https://www.cnblogs.com/xiaoxiaoqingyi/p/6676096.html)
[cmake知识](https://blog.csdn.net/bigdog_1027/article/details/79113342)

最近在学opencv 卡着报错 上面参考链接这个老哥太牛逼了 我写博客其实都是为了给自己记录用 与别人无关 那些写博客引人入胜的人 实在厉害
1. 烦人的bug
  opencv-lib.so not found 这个bug卡的我 一直在改cmake文件配置 结果并不是配置出现问题
  参考上面链接那位大哥的说法 需要添加一下配置
  ```
  build.gradle ... 
    android ....
    externalNativeBuild {
        cmake {
            cppFlags ""
            arguments "-DANDROID_STL=c++_shared"//使用c++_shared.so
        }
    }
  ```