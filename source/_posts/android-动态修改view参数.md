---
title: android 动态修改view参数
date: 2019-10-26 22:23:48
tags:
---
动态修改view参数的几种方法
[刘皇叔的博客](https://liuwangshu.cn/application/view/2-sliding.html)
<!-- more -->
1. `view.layout()`
```java
public void size1() {
    layout(10, 10, 100, 100);
}
```
2. `offsetLeftAndRight()`与`offsetTopAndBottom()`
```java
public void size2() {
    offsetLeftAndRight(10);
    offsetTopAndBottom(10);
}
```
参数是10 代表向右向下各移动`10px`

3. 通过改变 `layoutParams` 参数
```java
public void size3() {
    LayoutParams layoutParams = getLayoutParams();
    layoutParams.width = 10;
    setLayoutParams(layoutParams);
}
```
因为`setLayoutParams`方法内部调用了`requestLayout` 所以设置完`layoutParams`后 就自动重新布局了

4. `scrollTo` && `scrollBy` && `Scroller`
这三个方法是一个模式的 `scrollBy` 内部调用了 `scrollTo`, `Scroller` 需要搭配前面两个方法才起作用。