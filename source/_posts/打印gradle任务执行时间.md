---
title: 打印gradle任务执行时间
date: 2021-02-03 21:49:20
tags: gradle
---
注册gradle listener 监听任务构建时间
<!-- more -->

添加在setting.gradle 文件下
- 通过TaskExecutionListener监听任务
- 通过BuildListener控制输出log时机
```groovy

class TaskTimeInfo {
    long total
    String path
    long start
    long end
}

class TaskTimeListener implements TaskExecutionListener, BuildListener {

    LinkedHashMap<String, TaskTimeInfo> timeCostMap = new LinkedHashMap<>()


    @Override
    void buildStarted(Gradle gradle) {
//        println("--buildStarted")
    }

    @Override
    void settingsEvaluated(Settings settings) {
//        println("--settingsEvaluated")
    }

    @Override
    void projectsLoaded(Gradle gradle) {
//        println("--projectsLoaded")
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
//        println("--projectsEvaluated")
    }

    @Override
    void buildFinished(BuildResult result) {
//        println("--buildFinished")
        timeCostMap.each { info ->
            println("${info.getKey()}  [${info.getValue().total}ms]")
        }
    }

    @Override
    void beforeExecute(Task task) {
//        println("--beforeExecute  >>  $task.name")

        TaskTimeInfo timeInfo = new TaskTimeInfo()

        timeInfo.start = System.currentTimeMillis()
        timeInfo.path = task.getPath()
        timeCostMap.put(task.getPath(), timeInfo)
    }

    @Override
    void afterExecute(Task task, TaskState state) {
//        println("--afterExecute >>  $task.name")

        TaskTimeInfo timeInfo = timeCostMap.get(task.getPath())
        timeInfo.end = System.currentTimeMillis()
        timeInfo.total = timeInfo.end - timeInfo.start
    }
}

gradle.addListener(new TaskTimeListener())
```

###### 在官网上看到一种构建优化思路很好 在测试开发阶段 配置只打包一套资源配置 以此来减少构建时间
```groovy
android {
  ...
  productFlavors {
    dev {
      ...
      // The following configuration limits the "dev" flavor to using
      // English stringresources and xxhdpi screen-density resources.
      resConfigs "en", "xxhdpi"
    }
    ...
  }
}
```
