---
title: android升级适配记录
date: 2019-10-29 22:15:41
tags: android
---
- [Apache HTTP 客户端弃用](https://developer.android.google.cn/about/versions/pie/android-9.0-changes-28?hl=zh_cn#apache-p)
android6.0 android 9.0  变更 
项目里用到了`Volley`需要使用`Apache Http Client`
1. 在清单文件`application`下添加 `<uses-library android:name="org.apache.http.legacy" android:required="false"/>`
2. 在build.gradle文件里添加 `android {useLibrary 'org.apache.http.legacy'}`

<!-- more -->

- [动态权限申请RxPermissions](https://github.com/tbruyelle/RxPermissions)
 android6.0变更

- [打开相册](https://www.jianshu.com/p/7c6a53db8b12)
项目里使用的是`ACTION_GET_CONTENT`获取本地文件 跳转的时候不需要特殊权限 返回的是文件的`URI` 从`URI`提取绝对地址需要根据判断来区分[UriUtils](https://gist.github.com/tangtingyun/89c7da4d66e6f41b388ac00581f7ce23)

- FileProvider适配
android7.0变更
在调用相机的时候 因为相机应用和你的应用是两个进程 需要通过fileProvier转换数据
```java
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    Uri uri ;
    if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.M){
        uri = Uri.fromFile(mCurrentPhotoFile);
    }else {
        uri =  FileProvider.getUriForFile(this,getString(R.string.provider_str),mCurrentPhotoFile);
        //加入uri权限 要不三星手机不能拍照
        List<ResolveInfo> resInfoList = getPackageManager().queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY;
        for (ResolveInfo resolveInfo : resInfoList) {
            String packageName = resolveInfo.activityInfo.packageName;
            grantUriPermission(packageName, uri, Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
        }
    }
    intent.putExtra(MediaStore.EXTRA_OUTPUT,uri);
    tartActivityForResult(intent, cameraCode);
```
- [https配置说明](https://developer.android.google.cn/training/articles/security-ssl?hl=zh_cn)
android9.0变更
假如是正规CA 只需要把url加上s即可 否则是以下几种情况
1. 颁发服务器证书的CA未知
2. 自签名证书
3. 中间环节问题
这种情况要自己做验证

- [配置信任http](https://developer.android.google.cn/training/articles/security-config?hl=zh_cn)
在清单文件application里面配置文件地址
```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest ... >
        <application android:networkSecurityConfig="@xml/network_security_config"
                        ... >
            ...
        </application>
    </manifest>
```
network_security_config.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!--允许所有网址使用非安全连接-->
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```
- [通知渠道](https://developer.android.google.cn/training/notify-user/channels.html#java)
android8.0变更

- [应用安全最佳做法](https://developer.android.google.cn/topic/security/best-practices?hl=zh_cn)

- [AlarmManager使用](https://juejin.im/entry/588628e8128fe10065eb62a9)

- [currentTimeMillis uptimeMillis elapsedRealtime 区别](https://developer.android.com/reference/android/os/SystemClock.html#setCurrentTimeMillis(long))

- [Volley信任https](https://stackoverflow.com/questions/2012497/accepting-https-connections-with-self-signed-certificates/3904473#3904473)
```java
HostnameVerifier hostnameVerifier = org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER;
DefaultHttpClient client = new DefaultHttpClient();
SchemeRegistry registry = new SchemeRegistry();
SSLSocketFactory socketFactory = SSLSocketFactory.getSocketFactory();
socketFactory.setHostnameVerifier((X509HostnameVerifier) hostnameVerifier);
registry.register(new Scheme("https", socketFactory, 443));
SingleClientConnManager mgr = new SingleClientConnManager(client.getParams(), registry);
sClient = new DefaultHttpClient(mgr, client.getParams());
// Set verifier
HttpsURLConnection.setDefaultHostnameVerifier(hostnameVerifier);
//传入HttpClientStack，才能保证cookies与服务器交互，默认volley无cookies支持。
queue = Volley.newRequestQueue(context, new HttpClientStack(sClient));
```

- android studio 配置阿里云代理
 [参考文章](https://www.jianshu.com/p/b038bd95444b)
 [参考文章](http://dkaishu.com/2019/06/01/Android-Studio-%E4%B8%AD%E9%85%8D%E7%BD%AE%E9%98%BF%E9%87%8C%E4%BA%91%E5%85%AC%E5%85%B1%E4%BB%A3%E7%A0%81%E5%BA%93(%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93)/)
 [阿里云镜像](https://help.aliyun.com/document_detail/102512.html?spm=a2c40.aliyun_maven_repo.0.0.361830543mJYeQ#h2-u914Du7F6Eu6307u53572)
 [init.gradle](https://docs.gradle.org/current/userguide/init_scripts.html)