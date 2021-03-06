---
title: ProtocolBuffer学习
date: 2021-03-04 20:26:20
tags:  Android
---

在Android中使用 Protocol Buffer
<!-- more -->

## Protobuf是什么

Protobuf是一种灵活高效可序列化的数据协议，支持多种语言，只需定义好数据结构，利用
Protobuf框架生成源代码，就可以很轻松地实现数据结构的序列化和反序列化



缺点: Protobuf采用了二进制格式进行编码，可读性差 缺乏自描述，如果不配合proto结构体根本看不出来
什么。

.proto示例
```
syntax = "proto3";
package me.me.me.protobuf;
option java_outer_classname = "LoginInfo";
message Login {
    string account = 1;
    string password = 2;
}
```

## Android中接入

1.在项目的根gradle配置如下

```
dependencies {
        classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.0'
}
```

2.在app的gradle中配置如下

```
apply plugin: 'com.google.protobuf'
android {
    sourceSets {
    main {
        // 定义proto文件目录
         proto {
                 srcDir 'src/main/proto'
                include '**/*.proto'
         } 
        }
    }
}
dependencies {
// 定义protobuf依赖,使用精简版
    implementation "com.google.protobuf:protobuf-lite:3.0.0"
    implementation ('com.squareup.retrofit2:converter-protobuf:2.2.0') {
        exclude group: 'com.google.protobuf', module: 'protobuf-java'
    }
}

protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.0.0'
    }
    plugins {
        javalite {
            artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'
        }
    }
    generateProtoTasks {
        all().each { task ->
            task.plugins {
                javalite {}
            }
        }
    }
}

```

然后再 src/main/proto目录下编写.proto文件 编写完成后重新build 让插件生成源代码


## 其它

- protobuffer 需要学习特定的语法知识 多个平台写一套就好了
- {"account": "wanger"} 对于这种数据 并不会把 `account` 传输过去 而是用定义在`.proto`文件中的标示符号 再做一定的运算 来传输
- protobuffer 采用变长编码传输 每次有效传输7位 剩下一位用来标示是否结束
- 既然支持多平台 也要注意在不同平台的细微差异性 前后端都是`java`开发还好 如果不一致是`php` `java` `node` 可能要多看文档 避免踩坑