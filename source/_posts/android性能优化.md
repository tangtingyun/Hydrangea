---
title: 启动优化小技巧
date: 2019-11-11 22:15:41
tags: android
---


1. log过滤 displayed 选择 No Filters可以看到系统提供的activity启动耗时日志 也可以方便看到当前页面对应那个activity   :)
<!-- more -->
2. traceView
  ```java
      // Starts recording a trace log with the name you provide. For example, the
    // following code tells the system to start recording a .trace file to the
    // device with the name "sample.trace".
    Debug.startMethodTracing("sample");
    ...
    // The system begins buffering the generated trace data, until your
    // application calls <code><a href="/reference/android/os/Debug.html#stopMethodTracing()">stopMethodTracing()</a></code>, at which time it writes
    // the buffered data to the output file.
    Debug.stopMethodTracing();
  ```
  生成 `sample.trace` 文件 使用as打开 查看具体方法调用

  一些概念
- Wall clock time：该时间信息表示实际经过的时间。
- Thread time：该时间信息表示实际经过的时间减去线程没有占用 CPU 资源的那部分时间。


*参考来源*
- [layout-inspector](https://developer.android.google.cn/studio/debug/layout-inspector)
- [Choreographer机制讲解](https://www.androidperformance.com/2019/10/22/Android-Choreographer/)