---
title: java中的char
date: 2021-03-24 21:25:35
tags:
---
关于java中的char理解
<!-- more -->

### Unicode字符平面映射

目前的Unicode字符分为17组编排，每组称为平面(Plane)，而每平面拥有65536个代码点. 然而目前只用了少数平面。

平面	   始末字符值	             中文名称	                 英文名称
0号平面	 U+0000 - U+FFFF	  基本多文种平面	    Basic Multilingual Plane，简称BMP
1号平面	 U+10000 - U+1FFFF	  多文种补充平面	    Supplementary Multilingual Plane，简称SMP
2号平面	 U+20000 - U+2FFFF	  表意文字补充平面	    Supplementary Ideographic Plane，简称SIP
3号平面	 U+30000 - U+3FFFF	  表意文字第三平面	    Tertiary Ideographic Plane，简称TIP
4号平面 至 13号平面	U+40000 - U+DFFFF	（尚未使用）	
14号平面	U+E0000 - U+EFFFF
15号平面	U+F0000 - U+FFFFF	
16号平面	U+100000 - U+10FFFF	

一个完整的 "字符" 是一个 code point；
一个code point可以对应到 1到2个code unit 
一个char 就叫一个code unit  用utf-16表示 范围是 0- 65536

所以BMP平面的字符 一个char 可以表示
如果超过BMP平面之外的字符  一个char无法表示 编辑器会提示 Too many characters in character literal

可以在java中Character的类定义中看到 最大值 最小值
```java
public static final char MIN_VALUE = '\u0000';
public static final char MAX_VALUE = '\uFFFF';
```

### Java字符串中每一个字符都对应一个char码
不是的
有些字符需要两个char来表示
```java
 char hahaChar = '😄'; //编译报错  Too many characters in character literal   一个char 无法表示BMP之外的字符
 String haha = "😄";
 System.out.println(haha);  // 😄
 System.out.println(haha.length()); // 2  因为需要用两个char 来表示
 System.out.println(haha.codePointCount(0, haha.length())); // 1  因为这其实是一个字符
```

参考资料:
- https://luan.ma/post/character-encoding/
- https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84
- https://blog.csdn.net/zxhoo/article/details/38819517
- https://www.jianshu.com/p/a7db6ac53d57
- https://www.zhihu.com/question/35937819
- https://www.zhihu.com/question/27562173
- https://qzy.im/blog/2020/07/how-many-bytes-does-char-occupy-in-java/
