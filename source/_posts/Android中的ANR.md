---
title: Android中的ANR
date: 2021-03-01 21:30:30
tags: Android
---
### ANR知识点学习和记录
<!-- more -->

ANR (Application Not Responding) 应用程序无响应. 如果你的应用程序在UI线程被阻塞太长时间,
就会出现ANR, 通常出现ANR, 系统会弹出一个提示框, 让用户知道, 该程序正在被阻塞, 是否继续等待还是关闭.

## ANR类型

出现ANR的一般有以下几种类型:
1. **KeyDispatchTimeout**
    input事件在5s内没有处理完成发生了ANR.
    locat日志关键字: `Input envent dispatching timed out`
    > Input的超时机制与其他的不同, 对于input来说即便某次事件执行时间超过timeout时长，只要用户后续没有再生成输入事件，则不会触发ANR

2. **BroadcastTimeout**
    前台: onReceiver在`10s`内没有处理完成
    后台: onReceiver在`60s`内没有处理完成 
    locat日志关键字: `Timeout of broadcast BroadcastRecord`

3. **ServiceTimeout**
    前台: `onCreate`, `onStart`, `onBind`等生命周期在`20s`内没有处理完成
    后台: `onCreate`, `onStart`, `onBind`等生命周期在`200s`内没有处理完成
    locat日志关键字: `Timeout executing service`

4. **ContentProviderTimeout**
    ContentProvider在`10s`内没有处理完成发生ANR.
    locat日志关键字: `timeout publishing content providers`

## 超时检测机制

AMS通知组件启动时埋下定时器, 组件启动成功 报告AMS 拆除定时器 否则超时爆炸。
> 对于Serivce, Broadcast, Input 发生ANR之后 最终都会调用AMS.appNotResponding
对于provider,在其进程启动时publish过程可能会出现ANR, 则会直接杀掉进程以及清理相应消息,
而不会弹出ANR的对话框

## ANR出现的原因

1. 主线程频繁进行耗时的IO操作
2. 多线程操作的死锁 主线程被block
3. 主线程被Binder对端block  (默认binder是同步调用 也有oneway方式的异步调用)
4. .....

## ANR分析

- 系统发生anr时 信息会记录在/data/anr/traces.txt文件中 
- 文件中记录了ANR发生时间, 包名 pid cpu实用情况 堆占用大小 还可以查看main线程调用栈尝试快速定位异常位置

## ANR监控方案

1. 仿系统`watchdog`在每隔一段时间向主线程post一个任务 检测任务是否顺利执行完成，来判断主线程是否卡顿 (其实感觉用blockcanary开源库 也可以)

2. 通过`FileObserver`观察/data/anr/目录  监听是否有trace文件写入.