---
title: Webview优化
date: 2021-03-07 09:52:35
tags: Android
---
日常Webview优化记录
<!-- more -->

## WebView的加载流程

- 用户感知的加载流程
  进入页面[空白] －> loading中[等待加载] －> 显示基本样式[无数据]
  -> 显示完成

- 程序层面的加载流程
  1. webView初始化   //无反馈
  2. 建立连接 接受页面 接受样式 脚本下载解析/执行 渲染 // 白屏
  3. 请求后端数据 接受数据 渲染 // loading
  4. 展现  // 完成
## WebView的初始化优化

当App首次打开时 默认并不是初始化浏览器内核的 
只有当创建WebView实例的时候 才会创建WebView的基础框架

- 方案一
  1. 在客户端刚启动时 就初始化一个全局的静态WebView 并且做好配置
  2. 当用户访问了web页面 动态的将这个webview添加到布局中使用
  3. 缺点: 额外的内存消耗 

- 方案二
WebView独立进程解决方案 独立的进程与主进程隔开 即使崩溃也互不影响

需要解决的问题
1. web端 和同进程的Activity 交互问题
     - java调用js 
       - api < 19  `loadurl`
       - api > = 19 `evaluateJavascript`
     - js调用java
       - 通过`addJavascriptInterface`在web端注入对象
2. web端 和主进程业务逻辑交互 跨进程
    - 涉及到跨进程通信 使用aidl 封装上叙操作 
