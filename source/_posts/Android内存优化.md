---
title: Android中的内存优化
date: 2021-03-08 21:44:13
tags: Android
---
内存优化琐碎记录
<!-- more -->

### Java对象内存布局
- 对象头
  - Mark Work  哈希码 分代年龄 锁状态标记
  - Class Pointer 指向类对象信息的指针
  - Length  数据长度  只有数组对象才有
- 实例数据
  - 包含对象所有成员变量，根据变量类型决定大小
  - boolean byte  1字节
  - short char  2字节
  - int float  4字节
  - long double  8字节
  - reference  8字节
- 对齐补充
  - 为了让对象的大小为8字节的整数倍

### 垃圾收集算法
- 标记清除算法
  - Mark-Sweep算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。
  - 缺点 效率略低 两边扫描 位置不连续 产生碎片（当需要分配较大对象时 没有连续内存 不得不再次GC）
- 复制算法
  - Copying 算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另一块上面，然后再把已使用过的内存空间一次清理掉
  - 实现简单 运行高效 没有内存碎片  利用率只有一半
- 标记整理算法
  - 标记整理算法采用和标记清除算法一样的方式进行对象的标记，但在清除时不同，在回收不存活的对象占用的空间后，会将所有的存活对象往空闲空间移动，并更新对应的指针。因此成本更高，但是解决了内存碎片的问题
  - 没有内存碎片 效率偏低  两遍扫描 指针需要调整
### Android内存分配与回收机制
- 内存分配
  Android的Heap空间是一个Generational Heap Memory的模型(这是个啥。。。 懒得查了 太术语)，最近分配的对象会存放在`Young Generation`区域，
  当一个对象在这个区域停留的时间达到一定程度，它会被移动到`old Generation`，最后累积一定时间再移动到`Permanent Generation`区域。
- Young Generation
  GC使用复制算法 (扫描出存活的对象，并复制到一块新的完全未使用的空间中)
  由一个Eden区和两个Survivor区组成，程序中生成的大部分新的对象都在Eden区中，当Eden区满时，还存活的对象将被复制到其中一个Survivor区，当此Survivor区满时，
  此区存活的对象又被复制到另一个Survivor区，当这个Survivor区也满时，会将其中存活的对象复制到年老代。
- Old Generation
  GC使用标记整理算法 (扫描出存活的对象，然后再回收未被标记的对象，回收后对空出的空间要么合并，要么标记出来便于下次分配，以减少内存碎片带来的损耗)
  一般情况下，年老代中的对象生命周期都比较长。
- Permanent Generation
  用于存放静态的类和方法，持久代对垃圾回收没有显著影响。

### GC root对象
- 静态变量 常量
- 局部变量表
- 本地方法栈中的 变量表
- 类加载器 Thread等

### Android低内存杀进程机制
Android基于进程中运行的组件及其状态规定了默认的五个回收优先级:
- Empty process 空进程
- Background process 后台进程
- Service process 服务进程
- Visible process 可见进程
- Foreground process 前台进程
AMS会对所有的进程进行评分(存放在变量adj中)，然后再将这个评分更新到内核，由内核去完成真正的内存回收( lowmemorykille).

### 常用的数据结构
ArrayList   新增删除最坏时间复杂度O(n) System.arraycopy()  查找快 随机查找 时间复杂度O(1)
LinkedList  java中的实现是双向链表  新增删除快 移动指针 查找需要遍历 查找最坏时间复杂度O(n)
HashMap     数组加链表/红黑树 有自动扩容机制 没有自动缩容机制 链表数大于8且总数量大于64 会转换为红黑树

- Android 平台特有数据结构
ArrayMap  SparseArray 针对数据量小 

- 参考资料
http://gaozhipeng.me/posts/arraymap/
http://jiezhi.github.io/2017/03/30/android-arraymap-vs-sparsearray/
http://gityuan.com/2019/01/13/arraymap/
https://proandroiddev.com/all-you-need-to-know-about-arraymap-sparsearray-49759c2ecbf9



###  内存三大问题
- 内存抖动
  - 内存波动图形呈 锯齿状 GC导致卡顿
- 内存泄漏
  - 在当前应用周期内不再使用的对象被GC Roots引用，导致不能回收，使实际可用内存变小
- 内存溢出
  - 即OOM，OOM时会导致程序异常。
  - Java堆内存溢出
  - 无足够连续内存空间
  - FD数量超出限制
  - 线程数量超出限制
  - 虚拟内存不足

### Android内存分析命令
- dumpsys meminfo
  - 查看进程的 oom adj
- procrank
  - 获取所有进程的内存使用排行榜
- cat /proc/meminfo
  - 查看内存信息
- free
  - 查看可用内存 数据来源于 /proc/meminfo
- showmap
  - 用于查看虚拟地址区域的内存情况
- vmstat
  - 周期性动态打印出进程运行队列，系统切换，CPU时间占比
### 常见分析工具
1. Memory Analyzer Tools  (MAT)
2. Memory Profiler AS自带
3. LeakCanary 使用弱引用队列检测对象是否被回收

### Android内存泄漏常见场景以及解决方案

- 资源性对象未关闭
  - 对于资源性对象不再使用时，应该立即调用它的close()函数，将其关闭，然后再置未null。
- 注册对象未注销
  - 例如动态广播，EventBus未注销造成的内存泄漏，我们应该在Activity销毁时及时注销。
- 类的静态变量持有大数据对象
  - 尽量避免使用静态变量存储数据，特别是大数据对象，建议使用数据库存储。
- 单例造成的内存泄漏
  - 优先使用Application的Context，如需使用Activity的Context，可以在传入Context时使用弱引用进行封装。
- 非静态内部类
  - 例如匿名handler，Message发出之后存储在MessageQueue中，在Message中存在一个target，它是Handler的一个引用，Message在Queue中存在的时间过长，就会导致Hander无法被回收。如果Hanlder是非静态的，则会导致外部Activity不会被回收，并且消息队列是在一个Looper中不断地轮询处理消息，当这个Activity退出时，消息还未来得及处理导致Activity资源无法及时回收，引发内存泄漏。
  - 解决方案 使用一个静态Handler内部类 以弱引用方式使用外部对象
  - 在外部对象生命周期结束时候，移除消息
- 容器中的对象没清理造成的内存泄漏
  - 在退出程序之前，将集合里的东西clear，然后置为null，再退出程序
- 动画
  - 在页面不可见时 取消动画