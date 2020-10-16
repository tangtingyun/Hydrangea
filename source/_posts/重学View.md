---
title: 重学View
date: 2020-10-16 13:17:12
tags:
---
 关于重学view绘制流程的想法
<!-- more -->
# 关于重学view绘制流程的想法

1. 不同viewgroup对待子view的测量算法不同 (从算法的角度来考虑)
    1. LinearLayout的高度算法是 垂直方向不断叠加子view的高度
    2. RelativeLayout的高度算法是指定子view里面最高的
2. 关于ViewParent
    1. requestLayout的精髓 在于找到ViewParent对象 向上调用
    2. 我们通俗理解的View 包含两类 容器类 和 原子类 由此来形成控件树
    3. 但是最顶层的DecorView 的 Parent是谁呢   就是 ViewRootImpl  在WindowManagerGlobal#addView的方法里调用ViewRootImpl#setView方法的时候assignParent了this 自此就可以控制三大绘制流程了
3. 关于MeasureSpec的获取 
    1. ViewGroup#getChildMeasureSpec  这个方法 就是经典的根据父容器的测量模式和子view要求的LayoutParams 来综合考虑 最终计算出子View的参考MeasureSpec
    2. View#resolveSizeAndState   相当于对wrap_content的修正
    3. View#getDefaultSize     这就是为什么自定义View的wrap_content会撑满父容器的理由 默认取了父容器的大小
4. 关于wrap_content的理解
    1. 感觉这个就像equals 或 hashCode 方法一样 对于wrap_content 系统无法确定你的大小到底是多少 需要开发者自定义这个行为 你要自己去测量这个模式
5. 对于measure layout draw
    1. layout 和 draw 应该不会执行多次 但 measure会  
    2. 有人把自定义view  比作从毛坯房到精装房的过程  我觉得还挺好的 就像室内设计师一样 你需要这里量量 那里量量 确定一下layout的位置 从这个角度想 measure多次 也就不过分了
    3. 对于draw来说  就是另一片天地啦 :)