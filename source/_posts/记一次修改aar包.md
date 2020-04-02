---
title: 记一次修改aar包
date: 2019-12-18 14:27:40
description:重打包arr 修复bug
tags:
---

<!-- more -->
1. [参考资料1](https://blog.csdn.net/stil_king/article/details/89574538)
2. [参考资料2](https://blog.csdn.net/q610098308/article/details/79738557)
客服小姐姐说应用会在android10上崩溃,我就去看问题
项目集成的有百度异常统计 去统计后台看日志 兰儿并没有 what? 因为公司没android10手机 只好在卡卡的机器上跑模拟器
定位下来是因为第三方sdk 在启动初始化的时候获取了imei号 可是android10上面返回了null,sdk拿到imei后请求了接口
okhttp -> addHeader, 因为value = null 直接异常了
因为这段代码在百度统计初始化前面 所以百度后台也没记录日志。
可是这是第三方sdk啊~ 我怎么办 并且这是一个aar包
无奈 之后修改jar包重打包 参考资料1满心欢喜的改完 无误的生成了class 一顿操作猛如虎 修改后缀名 -> 解压 -> 修改后缀名 -> 解压 -> 替换 -> ok 
然后替换项目里原有的aar, wha他?? 爆红了 cant resolve 什么鬼
一模一样啊 也正确依赖了 怎么报错
按参考资料2 压缩包不要解压出来 直接打开 拖  不过拖的时候也有个提示 我直接选默认的了 (猜测如果修改后缀名解压再压缩 应该破坏了一些文件信息)
然后再替换 竟然ok了 哈哈哈哈~~~

打完收工 上线啦 回去写坑坑的vue了
