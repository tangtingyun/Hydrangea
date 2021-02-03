---
title: 在Android中使用IconFont
date: 2020-08-28 16:04:33
tags: Andorid Icon
---
**最近工作中接触到IconFont的使用 做个小记录**

<!-- more -->

*写web的时候经常用到iconfont来实现小图标的使用 在Android中也可以使用*

在这里借用了第三方开源库

[Android-Iconics](https://github.com/mikepenz/Android-Iconics)


#### 集成

```
    // 不包含UI的 只要包含 IconDrawable
    implementation "com.mikepenz:iconics-core:${latestAndroidIconicsRelease}"
    // 扩展库 包含自定义TextView  ImageView 等
    implementation "com.mikepenz:iconics-views:${latestAndroidIconicsRelease}"
```

这样就具备使用iconfont的能力了  我并没有用官网提供的字体图标库 用的是阿里的矢量图标
在Android中使用不像web那样用的是 classname 来引用，Android中需要使用字符表示
关于这块参考了文章 [iconfont 在线预览工具及其解析](https://segmentfault.com/a/1190000020121850)
文中介绍 从阿里矢量图网站下载下来的unicode模式是 &#xe636; 的表示形式 

以[&#]开头的 后接十进制数
以[&#x]开头的 后接十六进制

我们需要使用[&#x]后面的数字 改为 '\u([&#x]后面的数字)' 的形式 并且我发现是大小写不敏感的  这样就可以使用了

另一点需要注意的是 设置的时候 需要用如下形式

```
view.text = "{fontId-iconName}"  //这样才能正确显示
```

#### 其它参考资料
- [Iconfont字体生成原理及使用技巧
原理](https://www.iconfont.cn/help/article_detail?article_id=1)
- [IconFont在Android中使用](https://www.jianshu.com/p/f872e2ff78d9)
- [多看官方例子但是这个项目要as3.6 才能打开 炸裂](https://github.com/mikepenz/Android-Iconics)

