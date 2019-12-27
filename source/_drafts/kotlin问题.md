---
title: kotlin问题
tags: jetpack
---

1. kotlin 集成 databinding 使用 BindingAdapter
    ```
         dataBinding {
            enabled true
        }
    ```
    除了官方文档这一句 还需要 
    ```
    apply plugin: 'kotlin-kapt'
    ```
    ```
     // kotlin 中使用静态方法 
        @BindingAdapter("app:imageUrl")
        fun ImageView.setImageUrl(url: String?) {
            url?.apply {
                if (url.isNotEmpty()) {
                    Glide.with(this@setImageUrl).load(url).into(this@setImageUrl)
                }
            }
        }
    ```
    我写完BindingAdapter之后 一直报错 imageUrl not find 鬼日的