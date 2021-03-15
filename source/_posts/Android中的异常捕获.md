---
title: Android中的异常捕获
date: 2021-03-14 13:23:28
tags: android
---
线上应用崩溃监控
<!-- more -->

## Java层崩溃捕获
  实现 Thread.UncaughtExceptionHandler接口 处理未捕获异常

  系统默认的实现在RuntimeInit.java类中
  ```
    // 202行
    Thread.setDefaultUncaughtExceptionHandler(new KillApplicationHandler(loggingHandler));
  ```
  系统默认会打印崩溃log日志

  我们自己做线上监控 可以重写`uncaughtException`方法 实现自己的日志收集
  伪代码：
  ```
    public void uncaughtException(Thread t, Throwable e){
        File file = new File("java_carsh.txt");
        try {
            PrintWriter pw = new PrintWriter(new FileWriter(file));
            pw.println("手机型号");
            pw.println("系统版本");
            pw.println("app版本号");
            pw.println("发生时间");
            .....
            e.printStackTrace(pw);
            pw.flush();
            pw.close();
        }catch(IOException e){

        }finally{
            // 获取默认的系统异常处理器 
           if(defaultUncaughtExceptionHandler != null) {
               defaultUncaughtExceptionHandler.uncaughtException(t,e);
           }
        }
    }
  ```
可以在application中开启后台线程将日志文件上传

## Native层崩溃捕获
##### as中ndk的配置
  ```
    defalutConfig{
        // 配置我们自己的C/C++代码要编译哪些CPU架构
        externalNativeBuild {
            cmake {
                abiFilters 'armeabi-v7a', 'x86'
            }
        }
        // 配置打包时会把哪些CPU架构打包进去
        ndk {
            abiFilters 'armeabi-v7a', 'x86'
        }
    }
    // 外面还有一个 和 defalutConfig里面那个不一样
    externalNativeBuild {
        cmake {
            // cmake文件地址
            path 'src/main/cpp/CMakeLists.txt'
        }
    }
  ```
##### Native层捕获原理
Linux信号机制 当应用程序运行异常时，Linux内核将产生错误信号并通知当前进程，一般的出现崩溃信号，Android系统默认缺省操作是直接退出我们的程序。但是系统允许我们给某一个进程的某一个特定信号注册一个相应的处理函数(signal), 因此NDK Crash的监控可以采用这种信号机制 自己处理异常。
|  信号    | 描述      |
| ---- | ---- |
| SIGSEGV     | 内存引用无效。     |
| SIGBUS     |  访问内存对象的未定义部分。    |
| SIGFPE     |  算术运算错误,除以零。    |
| SIGILL     |  非法指令,如执行垃圾或特权指令    |
| SIGSYS     |  糟糕的系统调用    |
| SIGXCPU     | 超过CPU时间限制。     |
| SIGXFSZ     | 文件大小限制。     |

##### 墓碑
Android本机程序本质上就是一个Linux程序，当它在执行时发生严重错误，也会导致程序崩溃，然后产生一个记录崩溃的现场信息的文件，而这个文件在Android系统中就是 `tombstones`墓碑文件，路径在`/data/tombstones`下面 普通应用无权限读取墓碑文件。
##### BreakPad 工具
谷歌的一个跨平台崩溃转储和分析的框架。breakpad在Linux中的实现就是借助Linux信号捕获机制实现的。
- 生成日志格式是 minidump 需要借助minidump_stackwalk解析 
- as安装目录 bin/lldb/bin 中有 minidump_stackwalk 工具
- 使用addr2line工具定位具体代码 需要使用有符号信息的so (debug环境的so)