---
title: Fragment记录
date: 2021-04-07 20:31:49
tags: Android
---

在Android中使用Fragment日常记录
<!-- more -->


#### Fragment基本使用

核心三步走：
1. 创建Fragment
2. 获取FragmentManager
3. 调用事务，添加，替换。

add remove replace 等只是记录操作 真正触发在commit里面
在androidx版本里面 `androidx.fragment.app` 生命周期的流转处理在: 
`FragmentManagerImpl#moveToState`  722行代码

事务最终的提交方法：
1. commit()
   1. 在主线程中异步执行，其实也是Hanlder抛出任务，等待主线程调度执行。
   2. commit需要在宿主Activity保存状态之前调用，否则会报错
2. commitAllowingStateLoss()
   1. 异步执行 
   2. 允许状态丢失
3. commitNow()
   1. 同步执行
   2. 也必须在Activity保存状态前调用，否则会抛出异常。
4. commitNowAllowingStateLoss()
   1. 同步执行
   2. 允许状态丢失

#### Fragment Fragmentmanager FragmentTransaction关系
- Fragment
  - 其实是对View的封装，它持有view，containerView，fragmentManager，childFragmentManager等信息。
- FragmentManager
  - 是一个抽象类，它定义了对一个Activity／Fragment中添加进来的Fragment列表，Fragment回退栈的操作，管理方法
  - 还定义了获取事务对象的方法
  - 具体实现在FragmentImpl中
- FragmentTransaction
  - 定义了对Fragment添加，替换，隐藏等操作，还有四种提交方法
  - 具体实现在BackStackRecord中
- 配图：
  - ![fragment相关类图.png](https://i.loli.net/2021/04/07/rL586loeFkjxinZ.png)
  
#### Activity和Fragment 的 onActivityForResult 回调

1. 当在Fragment里面调用startActivityForResult的时候 Activity和Fragment里面的onActivityForResult都会回调，只不过Fragment里面回调的requestCode是正确的
2. 当在Fragment里面调用getActivity().startActivityForResult的时候，就会回调Activity里面的onActivityForResult
3. 当Fragment存在多层嵌套的情况，内层的Fragment调用startActivityForResult的时候，onActivityForResult方法不回调，此时需要创建一个BaseActivity复写它的onActivityResult方法

```java
        public class BaseFragmentActiviy extends FragmentActivity {
        private static final String TAG = "BaseActivity";
        
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        FragmentManager fm = getSupportFragmentManager();
        int index = requestCode >> 16;
        if (index != 0) {
        index--;
        if (fm.getFragments() == null || index < 0
            || index >= fm.getFragments().size()) {
            Log.w(TAG, "Activity result fragment index out of range: 0x"
            + Integer.toHexString(requestCode));
            return;
        }
        Fragment frag = fm.getFragments().get(index);
        if (frag == null) {
            Log.w(TAG, "Activity result no fragment exists for index: 0x"
            + Integer.toHexString(requestCode));
        } else {
            handleResult(frag, requestCode, resultCode, data);
        }
        return;
        }
        
        }
        
        /**
        * 递归调用，对所有子Fragement生效
        * 
        * @param frag
        * @param requestCode
        * @param resultCode
        * @param data
        */
        private void handleResult(Fragment frag, int requestCode, int resultCode,
        Intent data) {
        frag.onActivityResult(requestCode & 0xffff, resultCode, data);
        List<Fragment> frags = frag.getChildFragmentManager().getFragments();
        if (frags != null) {
        for (Fragment f : frags) {
            if (f != null)
            handleResult(f, requestCode, resultCode, data);
        }
        }
        }
        // 启动Activity的时候一定要用根Fragment
        /**
        * 得到根Fragment
        * 
        * @return
        */
        private Fragment getRootFragment() {
        Fragment fragment = getParentFragment();
        while (fragment.getParentFragment() != null) {
        fragment = fragment.getParentFragment();
        }
        return fragment;
        
        }
        
        /**
        * 启动Activity
        */
        private void onClickTextViewRemindAdvancetime() {
            Intent intent = new Intent();
            intent.setClass(getActivity(), YourActivity.class);
            intent.putExtra("TAG","TEST"); 
            getRootFragment().startActivityForResult(intent, 1001);
        }
```

#### Fragment的getActivity方法返回null的原因：

如果系统内存不足、或者切换横竖屏、或者app长时间在后台运行，Activity都可能会被系统回收，但是Fragment并不会随着Activity的回收而被回收，从而导致Fragment丢失对应的Activity。这里，假设我们继承于FragmentActivity的类为MainActivity，其中用到的Fragment为FragmentA。
 app发生的变化为：某种原因系统回收MainActivity——FragmentA被保存状态未被回收——再次点击app进入——首先加载的是未被回收的FragmentA的页面——由于MainActivity被回收，系统会重启MainActivity，FragmentA也会被再次加载——页面出现混乱，因为一层未回收的FragmentA覆盖在其上面——（假如FragmentA使用到了getActivity()方法）会报NullPointerException.




#### ViewPager配合Fragment

- 预加载
ViewPager会默认预加载左右两侧的fragment，通过Viewpager#setOffscreenPageLimit可以设置 至少是 1
预加载是为了缓存 能更快的展示页面 可是我们的有时又不想在页面还没有展示的时候，就去获取资源。于是衍生出懒加载

- 懒加载
当页面真正显示的时候 才去请求资源

ViewPager 配合 PagerAdapter 显示页面
关键方法是 ViewPager#populate 分以下五步：
- 准备适配 
  - startUpdate        // 准备缓存初始化工作
- 创建适配的Item数据
  - instantiateItem   // 处理缓存数据，增加和删除缓存节点item信息
- 销毁适配的Item数据
  - destroyItem       // 处理缓存数据，增加和删除缓存节点item信息
- 设置当前显示的Item数据
  - setPrimaryItem    // 处理缓存数据，增加和删除缓存节点item信息
- 完成适配
  - finishUpdate       // 触发缓存 启动真正的数据Item更新业务


###### 生命周期配图

![viewPager中fragment的生命周期.png](https://i.loli.net/2021/04/07/bwjJCQVY8dhFgeK.png)


##### ViewPager下懒加载方案
// 仅适用于一层frament 如果又childFragment 更复杂。。。。
```java
      public abstract class LazyFragment extends Fragment {

          private View rootView = null;
          private boolean isViewCreated = false; // View加载了
          private boolean isVisibleStateUP = false; // 记录上一次可见的状态

          @Nullable
          @Override
          public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
              E("onCreateView: ");
              if (rootView == null) {
                  rootView = inflater.inflate(getLayoutRes(),container, false);
              }
              isViewCreated  = true; // 1#. 解决未初始化奔溃
              initView(rootView); // 初始化控件 findvxxx
              // 2#. 解决第一次一直初始化loading一直显示的问题
              if (getUserVisibleHint()) {
                  // 手动来分发下
                  setUserVisibleHint(true);
              }
              return rootView;
          }

          // 判断 Fragment 是否可见
          @Override
          public void setUserVisibleHint(boolean isVisibleToUser) {
              super.setUserVisibleHint(isVisibleToUser);
              E("setUserVisibleHint");
              if (isViewCreated) {
                  // 3#. 记录上一次可见的状态: && isVisibleStateUP
                  if (isVisibleToUser && !isVisibleStateUP) {
                      dispatchUserVisibleHint(true);
                  } else if (!isVisibleToUser && isVisibleStateUP){
                      dispatchUserVisibleHint(false);
                  }
              }
          }

          // 分发 可见 不可见 的动作
          private void dispatchUserVisibleHint(boolean visibleState) {
              // 记录上一次可见的状态 实时更新状态
              this.isVisibleStateUP = visibleState;
              if (visibleState) {
                  // 加载数据
                  onFragmentLoad();
              } else {
                  // 停止一切操作
                  onFragmentLoadStop();
              }
          }

          // 让子类完成，初始化布局，初始化控件
          protected abstract void initView(View rootView);
          protected abstract int getLayoutRes();

          // -->>>停止网络数据请求
          public void onFragmentLoadStop() {
              E("onFragmentLoadStop");
          }

          // -->>>加载网络数据请求
          public void onFragmentLoad() {
              E("onFragmentLoad");
          }

          @Override
          public void onResume() {
              super.onResume();
              E("onResume");
              // 4#. 不可见 到 可见 变化过程  说明可见
              if (getUserVisibleHint() && !isVisibleStateUP) {
                  dispatchUserVisibleHint(true);
              }
          }

          @Override
          public void onPause() {
              super.onPause();
              E("onPause");
              // 5#. 可见 到 不可见  变化过程  说明 不可见
              if (getUserVisibleHint() && isVisibleStateUP) {
                  dispatchUserVisibleHint(false);
              }
          }

          @Override
          public void onDestroyView() {
              super.onDestroyView();
              E("onDestroyView");
          }
          private void E(String  string) {
             Log.i("LazyFrament", "name: " + mFragment.getClass().getSimpleName() + " -> " + method);
          }
      }

```