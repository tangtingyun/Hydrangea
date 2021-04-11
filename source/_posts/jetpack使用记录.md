---
title: jetpack使用记录
date: 2021-04-09 21:44:58
tags: Android
---
记录自己学习jetpack的过程 哈哈 持续输出中....
<!-- more -->

Jetpack 是Google 为解决Android开发碎片化，打造成熟健康生态圈提出的战略规划，是Google对Android未来提出的发展方向。
Jetpack是众多组件库的统称，AnxroidX是这些组件的统一包名。

Jetpack在开发中的好处:
- Jetpack拥有基于生命周期感知的能力，可以减少NPE崩溃，内存泄漏。为我们开发出健壮且流畅的程序提供强力保障。
- Jetpac 可以消除大量重复样板式的代码，可以加速Android的开发进程，这些组件可搭配工作，也可单独使用，同时配合Kotlin语言特性能显著提高工作效率。
- 统一开发模式，抛弃传统的mvc，mvp。

Jetpack 各个版本细节不一样 我看的是这个版本:
```groovy
    def lifecycle_version = "2.3.0"
    def arch_version = "2.1.0"

    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // 老的版本不推荐了 
    // lifecycle-extensions 中的 API 已弃用。您可以为特定 Lifecycle 工件添加所需的依赖项。
```
## Lifecycle：具备宿主生命周期感知能力的组件

Lifecycle是具备宿主生命周期感知能力的组件，它能持有组件（比如Activity和Fragment）
生命周期状态信息，并且允许其他观察者监听宿主的状态。它也是Jetpack组件库的核心基础，包括LiveDate，ViewModel组件
等也都是基于它来实现的。
> 再也不用手动分发宿主生命周期，再也不用手动反注册了。

使用方法：
1. LifecycleObserver 配合注解
2. FullLifecycleObserver接口 拥有宿主所有生命周期事件 (外界不能使用 非public)
3. LifecycleEventObserver 宿主生命周期事件封装成了Lifecycle.Event
   
```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

###### Fragment是如何实现Lifecycle的

Fragment实现Lifecycle是通过在各个生命周期方法里利用LifecycleRegistry分发相应的事件给每个观察者

```java
    public class Fragment implements LifecycleOwner {
        LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
        @Override
        public Lifecycle getLifecycle() {
            return mLifecycleRegistry;
        }
        void performCreate(){
            mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
        }
        .....
        void performResume(){
            mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
        }
    }
```

###### Activity是如何实现Lifecycle的

Activity实现Lifecycle是接入ReportFragment往Activity上添加一个无页面的fragment用来报告生命周期的变化
ReportFragment就是一个普通的无页面Fragment，通过LifecycleRegistry派发相应的Lifecycle.Event事件给每个观察者。

```java
    package androidx.core.app;

    public class ComponentActivity extends Activity implements LifecycleOwner{
    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
    protected void onCreate(Bundle bundle) {
        super.onCreate(savedInstanceState);
    }
```

###### LifecycleOwner／Lifecycle／LifecycleRegistry的关系

- LifecycleOwner：Activity／Fragment都实现了该接口，用以声明它是一个能够提供生命周期事件的宿主，同时必须复写getLifecycle方法提供一个Lifecycle对象，你也可以自己实现一个LifecycleOwner。
- Lifecycle：是一个抽象类，定义了两个枚举 State 宿主的状态，Event需要分发的事件的类型。
- LifecycleRegistry：是Lifecycle的唯一实现类，主要用来负责注册Observer，以及分发宿主的状态事件给它们。

LifecycleOwner/Lifecycle/LifecycleRegistry关系_盗图:

![lifecycleowner_lifecycle.png](https://i.loli.net/2021/04/10/39tCFQHx7hmjWak.png)

生命周期和状态流转_官方图：

![Snip20210410_2.png](https://i.loli.net/2021/04/10/CmeYobV9NZDgF25.png)

###### 添加observer时 完整的生命周期事件被分发

基于Lifecycle 在任意生命周期方法内注册观察者都能接受到完整的生命周期事件，比如在onResuem中注册一个观察者，它依次会收到：
```
LifecycleEvent.onCreate -> LifecycleEvent.onStart -> LifecycleEvent.onResume
```
在observer被添加的时候 被ObserverWithState包装，并且初始化状态传入的是INITIALIZED，之后会
依次流转到宿主当前的生命周期

## LiveData: 具备生命周期感知能力的数据订阅，分发组件

LiveData组件是基于观察者的消息订阅／分发组件，具有宿主(Activity/Fragment) 生命周期感知能力，这种感知能力可
`确保LiveData仅分发消息给处于活跃状态的观察者，即只有处于活跃状态的观察者才能收到消息。`
> 活跃状态：通常情况下等于Observer所在宿主处于started，resumed状态，如果使用ObserverForever注册的化，则一直处于活跃状态。

LiveData的消息分发机制，是以往的Handler，EventBus等无法比拟的，它们不会顾及当前页面是否可见，一股脑的有消息就发。
导致即便`应用在后台页面不可见的情况下还在做一些无用的工作抢占资源`，LiveData的出现解决了以往使用callback回调可能带来
的NPE，生命周期越界，无用工作抢占资源等。

###### LiveData的用法

常用方法: 

- observe(LifecycleOwner owner, Observer observer)
  - 注册和宿主生命周期关联的观察者
- observeForever(Observer observer)
  - 注册观察者，不会反注册，需要自己维护
- serValue(T data)
  - 发送数据，没有活跃的观察者时不分发。只能在主线程
- postValue(T data)
  - 和setValue一样。 不受主线程限制
- onActive 
  - 当且仅当有一个活跃的观察者时会触发
- inActive
  - 不存在活跃的观察者时会触发 

相关类操作:

- MutableLiveData
  - postValue/setValue
- MediatorLiveData
  - 可以统一观察多个LiveData发射的数据进行统一的处理
  - 同时也可以作为一个LiveData，被其他Observer观察
- Transformations.map 操作符
  - 可以对Livedata的数据进行变化，并且返回一个新的Livedata对象

###### LiveData实现原理

黏性消息扽发流程，即新注册的observer也能接收到前面发送的最后一条数据(只能接收到最新的一条, 之后版本号就一致了). 原因在于开发者注册的observer内部被ObserverWrapper包装内部有mVersion变量标记当前数据的版本号, LiveData每发送一条数据它的mVersion都会 +1, 新注册的Observer默认mVersion等0 

关键类:
```java
  // 即持有了LiveData的 observer 同时可以监听生命周期事件 
 class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
        @NonNull
        final LifecycleOwner mOwner;

        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
            super(observer);
            mOwner = owner;
        }

        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
            if (currentState == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            Lifecycle.State prevState = null;
            while (prevState != currentState) {
                prevState = currentState;
                activeStateChanged(shouldBeActive());
                currentState = mOwner.getLifecycle().getCurrentState();
            }
        }

        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }

        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }
```

可以基于LiveData实现 EventBus类似时间总线功能，这样就不用多引入一个三方库啦.




## ViewModel: 具备生命周期感知能力的数据存储组件
## SavedState  
## Room： 数据库操作
## DataBinding：只是一种工具，它解决的是View和数据之间的双向绑定
## Paging： 列表分页组件，可以轻松完成分页预加载以达到无限滑动的效果
## Navigation： 统一路由组件
## WorkManager：后台任务管理组件
## 老项目适配Androidx