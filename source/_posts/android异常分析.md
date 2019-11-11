---
title: android异常分析
date: 2019-11-11 22:15:41
tags: android
---

- [google开发者中文网](https://developers.google.cn/china)
- [AOSP在线阅读国内镜像](http://aospxref.com/)

<!-- more -->

- [参考博客 java异常分析](https://blog.csdn.net/wwj_748/article/details/51520020)
- [参考博客 native异常分析](https://blog.csdn.net/wwj_748/article/details/51542359)
- [官网参考](https://developer.android.google.cn/studio/profile/memory-profiler)

预防
 1. LeakCanary
 2. lint检查

分析工具
 1. android studio Profile

生成文件:
  1. dump文件  
  2. trace文件  确保应用有权向外部存储写入数据 (WRITE_EXTERNAL_STORAGE)
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
  3. hprof文件  查看 Java 堆和内存分配