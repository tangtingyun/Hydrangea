---
title: Android中的Bitmap
date: 2021-03-23 22:19:04
tags:  Android
---

android中的bitmap是内存大户, 在这里做下平时开发Bitamp遇到的知识点

<!-- more -->

#### 如何计算图片占用内存的大小？
首先是计算 内存中的大小 注意是占用内存 不是文件大小

- 直接计算

官方文档 
https://developer.android.google.cn/training/multiscreen/screendensities
https://www.jianshu.com/p/3f6f6e4f1c88

|        |  MDPI   | HDPI  | XHDPI  | XXHDPI  | XXXHDPI  |
|  ----  | ----  | ----  |----  |----  | ----  |
| density     | 160 | 240 | 320 | 480 | 640 |
| densityDpi  | 1 | 1.5 | 2 | 3 | 4 |

view  ->  Canvas  -> device / device2.0 / device3.0 / .....

图片的来源:

assets   // Assets中的资源等同于文件系统
drawable      // drawable 目录系列 注意dpi类型的影响
drawable-hdpi
drawable-mdpi
drawable-xhdpi
drawable-xxhdpi
drawable-xxxhdpi
raw     //  raw中的资源不经过任何的处理

宽 112px 高 131px 格式 png/ARGB_8888  文件大小 20.57kb

假设设备密度是 3

在assets文件夹下  加载到内存  宽 112px 高 131px  大小57.3kb   112*131*4 ＝ 58688 B
drawable          1     图片原始宽高 除以  1 (所在文件夹密度)  在乘于 3 (屏幕密度)
drawable-hdpi     1.5
drawable-mdpi     1
drawable-xhdpi    2
drawable-xxhdpi   3
drawable-xxxhdpi  4
drawable-nodpi    不会有缩放 原始图片多大就是多大

最终加载图片的大小：图片原始宽高 除以  所在文件夹密度  再乘于 屏幕密度
对应代码
https://cs.android.com/android/platform/superproject/+/master:frameworks/base/libs/hwui/jni/BitmapFactory.cpp;drc=master;bpv=1;bpt=0;l=215

```c++
    // BitmapFactory.Options#isScaled 默认是 true
    if (env->GetBooleanField(options, gOptions_scaledFieldID)) {
            const int density = env->GetIntField(options, gOptions_densityFieldID);
            const int targetDensity = env->GetIntField(options, gOptions_targetDensityFieldID);
            const int screenDensity = env->GetIntField(options, gOptions_screenDensityFieldID);
            if (density != 0 && targetDensity != 0 && density != screenDensity) {
                scale = (float) targetDensity / density;
          }
    }
```

看到一篇好牛皮的文章 我现在的水平还吃不懂
https://www.sunmoonblog.com/2019/06/15/bitmap-creation/


- 可以运行时获取
  - Bitmap#getByteCount()  // 图片占用内存大小的理论需求值
  - Bitmap#getAllocationByteCount() // 图片实际占用的内存大小 因为复用的时候 可以复用比自己宽高大的图片


#### 如何跨进程传递大图片
性能  减少拷贝次数
内存泄漏 资源及时关闭

- 给图片保存到固定的地方 传key给对方
  
- 通过IPC的方式转发图片数据
  - Binder      大小限制
  - Socket 管道  大小限制
  - 共享内存

TransactionTooLargeException
- 发出去的或者返回的数据量太大
- Binder缓存用于该进程所有正在进行中的Binder事务

intent 传递容易抛出异常
```
Bundle b = new Bundle()  // bitmap 太大会抛出异常
b.putParcelable("bitmap", mBitmap);
intent.putExtras(b)
```

使用Binder传  底层ashmem机制
```
Bundle b = new Bundle();
b.putBinder("binder", new IRemoteCaller.Stub(){
  public Bitmap getBitmap(){
    return mBitmap;
  }
})
intent.putExtras(b);
```

如果不是图片 可以考虑
ContentProvider
MemoryFile



#### 图片内存体积优化
- 跟文件存储格式无关
- 使用inSampleSize采样：大图 －》 小图  缩略图 
- 使用矩阵变换来放大图片：小图 －》 大图
- 使用RGB_565来加载不透明图片
- 使用.9图片做背景 
- 不使用图片 代码写动画
- 使用矢量图
- 使用icon-font
  
#### 索引模式 Indexed Color


#### Bitmap的使用
- 尽量根据实际需求选择合适的fenbianlv
- 注意原始文件分辨率与内存缩放的结果
- 不用桢动画，使用代码实现动效
- 考虑对Bitmap的重采样和复用配置
  
#### 配置信息与压缩方式
Bitamp中有两个内部枚举类
- Config 是用来设置颜色配置信息
- CompressFormat 是用来设置压缩方式

###### Config配置
- ALPHA_8      
  > 颜色信息只由透明度组成 占8位
- ARGB_4444    
  > 颜色信息由rgba四部分组成，每个部分都占4位，总共占16位。 被废弃
- ARGB_8888
  > 颜色信息由rgba四部分组成，每个部分都占8位，总共占32位。是Bitmap默认的颜色配置信息。
- RGB_565
  > 颜色信息由rgb三部分组成，R占5位，G占6位，B占5位，总共占16位。
- RGBA_F16
  > Android8.0新增 更丰富的色彩表现HDR 占64位 8个字节
- HARDWARE
  > Anroid8.0新增 Bitmap直接存储在graphic memory 

通常我们优化Bitmap会使用 Config.RGB_565这个配置

###### CompressFormat配置

- .JPEG   有损压缩  不支持透明度
- .PNG    无损压缩
- .WEBP

#### 高效加载大型位图

###### 通过降低采样率
  
https://developer.android.google.cn/topic/performance/graphics/load-bitmap

在解码时将 inJustDecodeBounds 属性设置为 true 可避免内存分配，为位图对象返回 null，但设置 outWidth、outHeight 和 outMimeType。

将按比例缩小的版本加载到内存中 考虑以下因素
  - 估算完整图片加载到内存的使用量
  - 图片要载入目标view的实际尺寸
  - 当前设备的屏幕大小和密度

通过评估 设置合适的采样参数

###### 分块加载

使用 BitmapRegionDecoder 分块加载 配合手势显示绘制区域

###### 开启复用

BitmapFactory.Options#inMutable = true 开启复用

BitmapFactory.Options#inBitmap  设置复用的bitmap对象

#### 缓存位图

###### 缓存算法
- 哪些应该保存
- 哪些应该丢弃
- 什么时候丢弃
- 获取成本
- 缓存成本
- 命中率
- 闭环验证 统计监控

###### 使用内存缓存
LruCache
LfuCache


###### 使用磁盘缓存
DiskLruCache


