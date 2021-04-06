---
title: Android中的图片压缩
date: 2021-04-06 20:26:20
tags: Android
---
[LuBan](https://github.com/Curzibn/Luban)压缩开源库探究
<!-- more -->

现在的手机像素越来越高，随便一张图片都是好几M，甚至几十M，这样的照片加载到app，可想而知，随便加载几张图片，手机内存就
不够用了，自然而然就造成了OOM，所以图片压缩异常重要，记录下自己对Luban开源框架的学习和探究

#### Android中的常用压缩

- 质量压缩
```java
    // 质量压缩 降低图片的质量，像素不会减少
    // 可以设置 0 - 100 来实现压缩 我看最新版的luban里设置的是 60
    bmp.compress(Bitmap.CompressFormat.JPEG,  10, baos);
```
- 尺寸压缩
```java
// 通过降低宽高采样 缩小图片大小来减少图片占用内存
  /**
         * If set to a value > 1, requests the decoder to subsample the original
         * image, returning a smaller image to save memory. The sample size is
         * the number of pixels in either dimension that correspond to a single
         * pixel in the decoded bitmap. For example, inSampleSize == 4 returns
         * an image that is 1/4 the width/height of the original, and 1/16 the
         * number of pixels. Any value <= 1 is treated the same as 1. Note: the
         * decoder uses a final value based on powers of 2, any other value will
         * be rounded down to the nearest power of 2.
         */
    public int inSampleSize;
```
- 使用libjpeg库 实现native哈夫曼压缩

#### LuBan

luban有两个分支 `main` 和 `turbo` 

`main` 分支使用的是java代码压缩

`turbo` 分支使用的是 `libjpeg-turbo` 并开启了 哈夫曼压缩 会添加额外的so包 看情况取舍吧
[c层实现](https://github.com/Curzibn/Luban-Turbo/blob/master/app/src/main/cpp/turbo.c)


关键类:

-Checker: 
 判断图片的工具
```java
  // 并没有简单的通过后缀名来判断图片类型 通过读取字节流中的文件标示
  private final byte[] JPEG_SIGNATURE = new byte[]{(byte) 0xFF, (byte) 0xD8, (byte) 0xFF};

  private boolean isJPG(byte[] data) {
    if (data == null || data.length < 3) {
      return false;
    }
    byte[] signatureB = new byte[]{data[0], data[1], data[2]};
    return Arrays.equals(JPEG_SIGNATURE, signatureB);
  }


  // 判断是否需要压缩 leastCompressSize默认是100KB 可以开发者自己设置
  boolean needCompress(int leastCompressSize, String path) {
    if (leastCompressSize > 0) {
      File source = new File(path);
      // https://blog.csdn.net/u014143369/article/details/53164791  获取文件大小的几种方法
      return source.exists() && source.length() > (leastCompressSize << 10);
    }
    return true;
  }

```

- Engine
  真正做压缩的方法
```java 
    // 设置inJustDecodeBounds为true 获取图片原始宽高
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    options.inSampleSize = 1;
    BitmapFactory.decodeStream(srcImg.open(), null, options);
    this.srcWidth = options.outWidth;
    this.srcHeight = options.outHeight;

　  // 计算出合理的inSampleSize 采样率
    private int computeSize() {
        srcWidth = srcWidth % 2 == 1 ? srcWidth + 1 : srcWidth;
        srcHeight = srcHeight % 2 == 1 ? srcHeight + 1 : srcHeight;

        int longSide = Math.max(srcWidth, srcHeight);
        int shortSide = Math.min(srcWidth, srcHeight);

        float scale = ((float) shortSide / longSide);
        if (scale <= 1 && scale > 0.5625) {
        if (longSide < 1664) {
            return 1;
        } else if (longSide < 4990) {
            return 2;
        } else if (longSide > 4990 && longSide < 10240) {
            return 4;
        } else {
            return longSide / 1280 == 0 ? 1 : longSide / 1280;
        }
        } else if (scale <= 0.5625 && scale > 0.5) {
        return longSide / 1280 == 0 ? 1 : longSide / 1280;
        } else {
        return (int) Math.ceil(longSide / (1280.0 / scale));
        }
   }

    // 真正开始执行压缩
     File compress() throws IOException {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inSampleSize = computeSize();

        Bitmap tagBitmap = BitmapFactory.decodeStream(srcImg.open(), null, options);
        ByteArrayOutputStream stream = new ByteArrayOutputStream();

        if (Checker.SINGLE.isJPG(srcImg.open())) {
            tagBitmap = rotatingImage(tagBitmap, Checker.SINGLE.getOrientation(srcImg.open()));
        }
        // 压缩质量为60 需要透明度则压缩为PNG 否则是 JPEG
        tagBitmap.compress(focusAlpha ? Bitmap.CompressFormat.PNG : Bitmap.CompressFormat.JPEG, 60, stream);
        tagBitmap.recycle();

        FileOutputStream fos = new FileOutputStream(tagImg);
        fos.write(stream.toByteArray());
        fos.flush();
        fos.close();
        stream.close();

        return tagImg;
    }

```

- Luban 
  对外的门户类 构造者模式方便调用
```java
    Iterator<InputStreamProvider> iterator = mStreamProviders.iterator();
    // 循环执行外部传入的文件路径
    while (iterator.hasNext()) {
      final InputStreamProvider path = iterator.next();

      AsyncTask.SERIAL_EXECUTOR.execute(new Runnable() {
        @Override
        public void run() {
          try {
            mHandler.sendMessage(mHandler.obtainMessage(MSG_COMPRESS_START));

            File result = compress(context, path);

            mHandler.sendMessage(mHandler.obtainMessage(MSG_COMPRESS_SUCCESS, result));
          } catch (IOException e) {
            mHandler.sendMessage(mHandler.obtainMessage(MSG_COMPRESS_ERROR, e));
          }
        }
      });
     　// 执行完移除
      iterator.remove();
    }

```

关于libjpeg-turbo实现
skia  ->  libjpeg 
对标
retrofit -> okhttp

```C++
        #include <jni.h>
        #include <string>
        #include <android/bitmap.h>
        #include <android/log.h>
        #include <malloc.h>
        extern "C"
        {   //引入libjpeg-turbo头文件
        #include "jpeglib.h"
        }
        #define LOGE(...) __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
        #define LOG_TAG "david"
        #define true 1

        typedef uint8_t BYTE;

        // 写入压缩文件到本地
        void writeImg(BYTE *data, const char *path, int w, int h) {

            //jpeg引擎   绕过 skia  来实现压缩
            struct jpeg_compress_struct jpeg_struct;
            // 设置错误处理信息
            jpeg_error_mgr err;
            jpeg_struct.err = jpeg_std_error(&err);
            //  给结构体分配内存
            jpeg_create_compress(&jpeg_struct);

            FILE *file = fopen(path, "wb");
            //  设置输出路径
            jpeg_stdio_dest(&jpeg_struct, file);

            jpeg_struct.image_width = w;
            jpeg_struct.image_height = h;
            //    初始化  初始化
            //改成FALSE   ---》 开启hufuman算法
            jpeg_struct.arith_code = FALSE;
            jpeg_struct.optimize_coding = TRUE;
            jpeg_struct.in_color_space = JCS_RGB;

            jpeg_struct.input_components = 3;
            //    其他的设置默认
            jpeg_set_defaults(&jpeg_struct);
            // 第二个参数 quality 压缩质量
            jpeg_set_quality(&jpeg_struct, 20, true);
            jpeg_start_compress(&jpeg_struct, TRUE);
            JSAMPROW row_pointer[1];
            //    一行的rgb   颜色数据只有 rgb  忽略 a
            int row_stride = w * 3;
            while (jpeg_struct.next_scanline < h) {
                row_pointer[0] = &data[jpeg_struct.next_scanline * w * 3];
                jpeg_write_scanlines(&jpeg_struct, row_pointer, 1);
            }
            jpeg_finish_compress(&jpeg_struct);

            jpeg_destroy_compress(&jpeg_struct);
            fclose(file);
        }

        extern "C"
        JNIEXPORT void JNICALL
            Java_com_maniu_wechatimagesend_MainActivity_compress(JNIEnv *env, jobject instance,
                                                            jobject bitmap, jstring path_) {
            const char *path = env->GetStringUTFChars(path_, 0);
            // 获取宽高
            AndroidBitmapInfo bitmapInfo;
            AndroidBitmap_getInfo(env, bitmap, &bitmapInfo);
            BYTE *pixels;
            AndroidBitmap_lockPixels(env, bitmap, (void **) &pixels);
            int h = bitmapInfo.height;
            int w = bitmapInfo.width;

            BYTE *data,*tmpData;
            data= (BYTE *) malloc(w * h * 3);
            tmpData = data;
            BYTE r, g, b;
            int color;
            // 颜色数据 只设置rgb 忽略 a
            for (int i = 0; i < h; ++i) {
                for (int j = 0; j < w; ++j) {
                    color = *((int *) pixels);
                    r = ((color & 0x00FF0000) >> 16);
                    g = ((color & 0x0000FF00) >> 8);
                    b = ((color & 0x000000FF));
                    *data = b;
                    *(data + 1) = g;
                    *(data + 2) = r;
                    data += 3;
                    pixels += 4;
                }
            }
            AndroidBitmap_unlockPixels(env, bitmap);
            writeImg(tmpData, path, w, h);
            free(tmpData);
            env->ReleaseStringUTFChars(path_, path);
        }
```