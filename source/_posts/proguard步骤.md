---
title: proguard步骤
tags: java android
date: 2020-04-21 16:38:41
---

<!-- more -->

混淆不只是表面看到的把方法名 变量 等变短了 还有压缩，优化字节码的功能

```sequence
Title:proguard
Input code-> Shrunk code:shrink
Shrunk code->Optimized code:optimize
Optimized code->Obfusscated code:obfuscate
```
```sequence
Obfusscated code->Output code:preverify
```

## Entry points

    关键方法入口 需要keep的资源 比如反射

## Step

- shrink `根据入口 递归所有资源, 未使用的资源会被丢弃`
- optimization `不是入口点的资源 可能被重置为 private static final 或者 inline`
- obfuscation `重命名不是入口点的资源`
- preverification 