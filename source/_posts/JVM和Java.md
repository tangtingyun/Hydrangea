---
title: JVM杂记
date: 2021-03-25 21:51:02
tags: Java
---

JVM 是一种规范
<!-- more -->

文档信息：
jvms13
jls12

任何语言 ->  class文件格式 -> JVM运行

JVM
JRE = jvm + core lib
JDK = jre + development kit

类加载过程
- 加载
  - classloader
- 链接 linking
  - verification   验证文件是否符合JVM规定
  - preparation    静态成员变量赋默认值
  - Resolution     将类，方法，属性等符号引用解析为直接引用 常量池中的各种符号引用解析为指针，偏移量等内存地址的直接引用。
- Initializzing
  - 调用类初始化代码 <clinit>
  - 静态变量赋值为初始值

对象的创建过程
- class loading
- class linking ( verification, preparation, resolution)
- class initializing
- 申请对象内存
- 成员变量赋默认值
- 调用构造方法<init>
  - 成员变量顺序赋初始值
  - 执行构造方法语句

JMM
- 硬件层数据一致性
- 乱序问题
- volatile的实现细节
  - 字节码层面   ACC_VOLATILE
  - JVM层面     volatile内存区的读写都加屏障
  - OS和硬件层面  windows lock指令实现
- synchronized实现细节
  - 字节码层面   ACC_SYNCHRONIZED   monitorenter monitorexit
  - JVM层面     C C++调用了操作系统提供的同步机制
  - OS和硬件层面   x86: lock cmpxchg/xxx
