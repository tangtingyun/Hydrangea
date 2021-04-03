---
title: 应用启动和Context
date: 2021-04-03 17:52:40
tags: Android
---
重温Context和学习应用是怎么启动的
<!-- more -->

#### 应用进程是怎么启动的

![应用进程启动](https://i.loli.net/2021/04/03/oHlE7O9aunZ3ctD.png)

进程的启动方式 fork
```c++
if ((pid = fork()) < 0) {
    // error  发生错误了
} else if(pid == 0) {
  // child process 在子进程中
  // 如果执行 execve 子进程资源被path替代
  // 否则 共享父进程资源
  execve(path, argv, env);
} else {
  // parent process 在父进程中
}
```

启动组件的时候 发现进程还没有启动 就去启动进程
```c++
ProcessRecord app = getProcessRecordLocked();
if(app != null && app.thread != null) {
    // 进程已经启动了 这里可以启动组件了
    return;
}
// 启动进程 或者等待正在启动的进程来报告
startProcessLocked(r.processName);
```

AMS和应用的双向调用 
![AMS和应用之间的调用](https://i.loli.net/2021/04/03/KFM3LWlntpg1kCs.png)

应用启动入口函数 ActivityThread#main
```java
public static void main(String[] args) {
    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    // 应用启动成功后向AMS打报告 注册ApplicationThread到AMS中
    thread.attach(false);

    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited")
}
```

#### Application
Application的作用
- 保存应用进程内的全局变量 不推荐
  - 内存不足 应用重建 静态变量没有被初始化
- 初始化操作
- 提供应用上下文

多进程中会有多个Application

Application 继承关系
```java
class Application extends ContextWrapper {}

class ContextWrapper extends Context {
    Context mBase; // 静态代理模式

    public ContextWrapper(Context base) {
        mBase = base;
    }

    protexted void attachBaseContext(Context base) {
        mBase = base;
    }
}
```

###### Application的生命周期

bindApplication ->  H.BIND_APPLICATION -> handleBindApplication
-> makeApplication -> callApplicationOnCreate

调用顺序如下:

![Application执行顺序](https://i.loli.net/2021/04/03/RklHB6XwDj3dmft.png)

- 构造函数
  - makeApplication -> ContextImpl.createAppContext() ->  使用classloader加载 反射创建Application
- attachBaseContext
  - 赋值 mBase
- onCreate
- onTerminate 只在模拟器中调用 真实环境不起作用

不要在构造函数里使用上下文 因为还没有准备好


#### Context

```java 
/**
 * Interface to global information about an application environment.  This is
 * an abstract class whose implementation is provided by
 * the Android system.  It
 * allows access to application-specific resources and classes, as well as
 * up-calls for application-level operations such as launching activities,
 * broadcasting and receiving intents, etc.
 */
public abstract class Context {  // 抽象类
    public abstract Resources getResources();
    public abstract Object getSystemService(String name);
    public abstract void startActivity(Intent intent);
    public abstract void sendBroadcast(Intent intent);
    .......
}
// 实现类
class ComtextImpl extends Context {
    final ActivityThread mMainThread;
    final LoadedApk mPackageInfo;

    private final ResourcesManager mResourcesManager;
    private final Resources mResources;

    private Resources.Theme mTheme = null;
    private PackageManager mPackageManager;

    final Object[] mServiceCache = 
        SystemServiceRegistry.createServiceCache();
    .......
}
```

Context 是在哪里创建的?

有自己的Context的组件:
- Application 
  - handleBindApplication -> makeApplication -> ContextImpl.createAppContext() 创建Application的Context -> application.attach(context) -> attachBaseContext 设置mBase
  - ![Application总结](https://i.loli.net/2021/04/03/6vgRxlfzUXD4QKC.png)
- Activity
  - performLaunchActivity() -> createBaseContextForActivity() //new ContextImpl()  -> activity.attach(appContext, ...)
  - ![Activity类继承](https://i.loli.net/2021/04/03/Dp6LnQHZGYcIatu.png)
  - ![Activity总结](https://i.loli.net/2021/04/03/WLe6GC9KjstSNZR.png)
- Service
  - ![Service总结](https://i.loli.net/2021/04/03/qvo8wj2NkYlJQGy.png)


没有自己的Context组件:
- BroadcastReceiver中的Context
  - 动态广播  就是注册时候的context.registerReceiver
  - 静态广播  是以Application为mBase的ContextWrapper
- ContentProvider中的Context
  - 传入的是Application的Context
  - ContextProvider的onCreate比Application的onCreate早











