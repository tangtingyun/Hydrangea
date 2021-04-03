---
title: Android中的View
date: 2021-04-01 21:32:53
tags:  Android
---
记录Android中View的显示和屏幕刷新机制
<!-- more -->

#### Activity的页面层级
- Activity
  - PhoneWindow        
    > 在Activity的attach中创建PhoneWindow

    - DecorView       
      > 把DecorView添加到WindowManager中  创建了ViewRootImpl对象 调用view.assignParent(this); 把ViewRootImple设置为DecorView的parent

      - ContentView


#### Activity的显示原理
在 handleResumeActivity 中 

先执行 onResume 生命周期 ——》 然后 vm.addView(decor, l); ——》 activity.makeVisible() 触发重绘;

在addView中 创建ViewRootImpl 然后调用 ViewRootImpl#setView 方法 

在setView方法中的两个重要步骤
- requestLayout
  - scheduleTraversals()
- mWindowSession.addToDisplay(mWindow)

一个ViewRootImpl 管理一个view tree 
在ViewRootImpl中有一个W类 实现了IWindow 传递到WMS来实现双向调用 


#### View中的parent
点击按钮触发requestLayout 会一层一层的往上触发重绘
这个过程就是不断的调用 view.parent.parent.parent....requestLaout()
那么最顶层的view的parent是谁呢？ 就是ViewRootImpl 它实现了ViewParent

没错就是在ViewRootImpl#setView方法中设置的
view.assignParent(this);  // 第4160行


#### Android中的屏幕刷新机制
requestLayout 只是往 Choreographer 中插入来一个 Runnable 并没有马上执行三大绘制流程
会在下一次vsync信号来的时候执行

![应用申请刷新流程](https://i.loli.net/2021/04/01/m6bFZ4CcJqS9wBR.png)

requestLayout之后的scheduleTraversals 先插入来一个同步屏障 
然后往Choreographer中post了一个callback
在post一个callback之后 会调用scheduleVsyncLocked  
这里Choreographer会去向surfaceFling申请下一次vsync信号
这样在下一次vsync信号来的时候就会通知Choreographer开始处理各种callback
INPUT
ANIMATION
TRAVERSAL
COMMIT

#### VSync信号机制
![Vsync信号的流转](https://i.loli.net/2021/04/01/Kfxj67HwXvLSFpB.png)


#### 在子线程更新UI的方法

对于View来说  它的UI线程就是创建ViewRootImpl时候的线程

- 在主线程申请完requestLayout后 在子线程做操作
  > 因为requestLayout有检测防抖机制 子线程可以趁这个空隙 一起更新掉
- 在子线程中创建ViewRootImpl
- 利用硬件加速机制绕开 requestLayout
  > 在硬件加速的支持下，如果控件只是调用了invalidate 而没有触发requestlayout是不会触发 ViewRootImpl#checkThread的。
- 使用SurfaceView控件
