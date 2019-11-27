---
title: leakcanary学习
tags:  源码学习
---
1. 使用
```groovy
Add LeakCanary to build.gradle:
dependencies {
  // debugImplementation because LeakCanary should only run in debug builds.
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.0-beta-3'
}
That’s it, there is no code change needed! LeakCanary will automatically 
show a notification when a memory leak is detected in debug builds.
```
官网说只用添加完依赖就可以自动开启监测, 比较好奇这点是怎么实现的 看了代码了解到是通过注册`ContentProvider`来实现自启动的 
意外学到一个知识点``ContentProvider 在`app`启动的时候 由系统自动安装的 触发了`onCreate`方法 `leakCanary`的初始化入口也在这里 
[ContentProvider分析](https://www.jianshu.com/p/3c81df444034)
[ContentProvider分析](https://blog.csdn.net/zhenjie_chang/article/details/62889188)
```kotlin
/**
 * Content providers are loaded before the application class is created. [AppWatcherInstaller] is
 * used to install [leakcanary.AppWatcher] on application start.
 */
internal sealed class AppWatcherInstaller : ContentProvider() {

  /**
   * [MainProcess] automatically sets up the LeakCanary code that runs in the main app process.
   */
  internal class MainProcess : AppWatcherInstaller()

  /**
   * When using the `leakcanary-android-process` artifact instead of `leakcanary-android`,
   * [LeakCanaryProcess] automatically sets up the LeakCanary code
   */
  internal class LeakCanaryProcess : AppWatcherInstaller() {
    override fun onCreate(): Boolean {
      super.onCreate()
      AppWatcher.config = AppWatcher.config.copy(enabled = false)
      return true
    }
  }

  override fun onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    InternalAppWatcher.install(application)
    return true
  }
  ....
}
```
2. 分析
源码拉下来后看到 分了很多层module 
leakcanary-object-warcher --> leakcanary-object-watcher-android --> leakcanary-object-watcher-android-androidX
2.1. leakcanary-object-warcher
    leakcanary-object-warcher实现了弱引用监测和gc的触发 包含以下类
    `Clock`
    `GcTrigger`
    `KeyedWeakReference`
    `ObjectWatcher`
    `OnObjectRetainedListener`
    关键代码就在`ObjectWatcher`里面
    如果`weakReference`被gc回收了就会放到对应的`ReferenceQueue`队列中，也就是说在队列里的 就表明内存被回收了 没有泄露 所以每次都会调用这个方法来检查是否被回收了
    ```kotlin
      private fun removeWeaklyReachableObjects() {
        // WeakReferences are enqueued as soon as the object to which they point to becomes weakly
        // reachable. This is before finalization or garbage collection has actually happened.
        var ref: KeyedWeakReference?
        do {
        ref = queue.poll() as KeyedWeakReference?
        // 如果已经被回收了 就从监控集合中移除掉 
        if (ref != null) {
            watchedObjects.remove(ref.key)
        }
        } while (ref != null)
      }
    ```
    `OnObjectRetainedListener`接口用来通知 有变量泄露了
    ```kotlin
        /**
        * Listener used by [ObjectWatcher] to report retained objects.
        */
        interface OnObjectRetainedListener {
        }
    ```
    当外界调用`watch`方法的时候
    ```kotlin
        /**
        * Watches the provided [watchedObject].
        *
        * @param name A logical identifier for the watched object.
        */
        @Synchronized fun watch(
            watchedObject: Any,
            name: String
        ) {
            //会多次调用这个方法 这点还没理解透彻 反正每次都是为了检查对象是否已经被会回收
            removeWeaklyReachableObjects()
            val key = UUID.randomUUID()
                .toString()
            val watchUptimeMillis = clock.uptimeMillis()
            // 生成KeyedWeakReference对象
            val reference = KeyedWeakReference(watchedObject, key, name, watchUptimeMillis, queue)
            // 保存到 watchedObjects 数组
            watchedObjects[key] = reference
            // 检查是否泄露    
            checkRetainedExecutor.execute {
                moveToRetained(key)
            }
        }
    ```
2.2 leakcanary-object-watcher-android分析
    那什么时候调用`watch`方法呢,在android层次的实现是
    `ActivityDestroyWatcher` -> `application.registerActivityLifecycleCallbacks(activityDestroyWatcher.lifecycleCallbacks)`
    ```kotlin
          private val lifecycleCallbacks =
            object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
                override fun onActivityDestroyed(activity: Activity) {
                    if (configProvider().watchActivities) {
                        // 在onActivityDestroyed回调里 观察activity是否泄露
                        objectWatcher.watch(activity)
                    }
                }
            }
    ```    
    这里有个`kotlin`的知识点
    [关键字by](https://kotlinlang.org/docs/reference/delegation.html#implementation-by-delegation) 上面这种写法
    应该是为了不想实现太多接口方法 所以写了一个通用实现，然后再复写方法 整个库都是kotlin重写的 歪果仁真是牛鼻
2.3 通过接口通知上层有内存泄漏时 触发`HeapDumpTrigger`相关逻辑最后通过`Debug.dumpHprofData(heapDumpFile.absolutePath)`生成`.dump`文件
    还通过`HeapAnalyzerService`后台服务分析`dump`文件将生成的结果通过页面展示出来 


[Java堆：Shallow Size和Retained Size](https://blog.csdn.net/kingzone_2008/article/details/9083327)