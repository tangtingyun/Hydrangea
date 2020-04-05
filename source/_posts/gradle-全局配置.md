---
title: gradle 全局配置
date: 2020-04-05 15:35:52
tags: gralde
---

gradle 全局 init.gradle 配置
<!-- more -->
 *受够了gradle构建慢 也受够了每次都手动添加国内镜像 决定一步到位*

 - [官方文档](https://docs.gradle.org/current/userguide/init_scripts.html)
 - [阿里云镜像文档](https://maven.aliyun.com/mvn/view)
 - [参考文章](https://blog.csdn.net/lj402159806/article/details/78422953)

    根据文档 可以在用户路径下.gradle文件夹下创建init.gradle 来指定全局配置

```groovy
buildscript {
    repositories {
        maven{ url 'https://maven.aliyun.com/repository/public'}
        maven { url 'https://maven.aliyun.com/repositories/jcenter' }
        maven { url 'https://maven.aliyun.com/repositories/google' }
        maven { url 'https://maven.aliyun.com/repository/central' }
    }
}

allprojects {
    repositories {
        maven{ url 'https://maven.aliyun.com/repository/public'}
        maven { url 'https://maven.aliyun.com/repositories/jcenter' }
        maven { url 'https://maven.aliyun.com/repositories/google' }
        maven { url 'https://maven.aliyun.com/repository/central' }
        maven { url "https://jitpack.io" }
    }
}
```   

ok 搞定 :)