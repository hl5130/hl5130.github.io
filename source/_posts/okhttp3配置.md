---
title: okhttp3配置
date: 2019-04-08 14:36:37
tags: okHttp3
categories: Android
---
# 依赖导入
在 ``module`` 的 ``build.gradle`` 文件中添加下面的代码，并重新build
```
implementation 'com.squareup.okhttp3:okhttp:3.14.0'
implementation 'com.squareup.okio:okio:2.2.2'
```

# Build 时出现的问题
如果 build 失败，并出现了下面的问题：
```
Caused by: com.android.builder.dexing.DexArchiveBuilderException: Failed to process /Users/hongliang/.gradle/caches/modules-2/files-2.1/com.squareup.okhttp3/okhttp/3.14.0/c3a9efd10f26a802da2c47dbe54a8d4c44a6019f/okhttp-3.14.0.jar
Caused by: com.android.builder.dexing.DexArchiveBuilderException: Error while dexing.
Caused by: com.android.tools.r8.CompilationFailedException: Compilation failed to complete
Caused by: com.android.tools.r8.utils.AbortException: Error: Static interface methods are only supported starting with Android N (--min-api 24): okhttp3.Request okhttp3.Authenticator.lambda$static$0(okhttp3.Route, okhttp3.Response)
```
## 原因
Java8之后才支持lambda和静态接口方法，所以需要指定使用Java8编译
## 解决办法
在 ``module`` 的 ``build.gradle`` 文件中添加下面的代码，并重新build
```
android {
  ...
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

# okHttp3系列文章
[okHttp3-基本使用](http://localhost:4000/2019/04/08/okhttp3基本使用)
