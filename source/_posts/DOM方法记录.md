---
title: DOM方法记录
date: 2019-10-25 21:53:47
tags:
---

<!-- more -->
1.`element.setAttribute(name, value);`
```js
.red {
    color: red;
}
var red = document.getElementById('red');
red.setAttribute('class', 'red')
```
最近在学习vue源码的时候 看到了这个方法 记录下