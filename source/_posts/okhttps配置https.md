---
title: okhttps配置https
date: 2019-10-19 11:10:49
tags: android
---
最近pos需要上hppts 需要配置 使用的是`ssl pinning` 我看了下理解就是把证书放到客户端本地读取验证 不走ca链 `ssl pinning` 又分公钥锁定和证书锁定[参考链接](https://www.infinisign.com/faq/what-is-ssl-pinning)

刚开是在网上找的文章后来放弃了 官方文档才是baba

- [官方https配置说明](https://square.github.io/okhttp/https/)
<!-- more -->

> ## Certificate Pinning
配置代码
```kotlin
  private val client = OkHttpClient.Builder()
      .certificatePinner(
          CertificatePinner.Builder()
              .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
              .build())
      .build()

  fun run() {
    val request = Request.Builder()
        .url("https://publicobject.com/robots.txt")
        .build()

    client.newCall(request).execute().use { response ->
      if (!response.isSuccessful) throw IOException("Unexpected code $response")

      for (certificate in response.handshake!!.peerCertificates) {
        println(CertificatePinner.pin(certificate))
      }
    }
  }
```
- [完整示例代码](https://github.com/square/okhttp/blob/ba2c676aaf2b825528955f61dd43004a5bd9ca98/okhttp/src/main/java/okhttp3/CertificatePinner.kt)

这个是官方给的`ssl pinning` 配置示例 但是我这样用没用 因为我怀疑我们公司用的是自签名证书 而`CertificatePinner.kt`里面有注释

`CertificatePinner can not be used to pin self-signed certificate if such certificate is not accepted by javax.net.ssl.TrustManager.`

> ## CustomTrust
采用官方第二种方式 ok了
- [完整示例代码](https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/CustomTrust.java)

这个示例代码里有行注释

`See also {@link CertificatePinner}, which can limit trusted certificates while still using the host platform's built-in trust store.`

 大概可以猜出来 `CertificatePinner` 还是需要官方ca链认证的 而我这种情况应该是自签名 所以第一种不行 第二种行 
