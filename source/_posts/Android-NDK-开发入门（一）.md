---
title: Android NDK 开发入门（一）
date: 2019-04-10 15:28:11
tags: NDK
categories: Android
---

# 简介
NDK开发适用于音频、视频、3D图形、相机驱动等开发。
## 开发工具简介
- Android NDK：是AndroidStudio的原生开发工具包，允许使用C和C++代码
- CMake：一款外部构建工具，可与Gradle搭配使用来构建原生库。如果只使用ndk-build，就不需要此组件
- LLDB：用来调试原生代码
## 开发工具下载
1. 打开 SDK Manager
2. 点击 SDK Tools 标签
3. 选中 LLDB、CMake 和 NDK
4. 点击 Apply，然后在弹出式对话框中点击 OK
5. 安装完成，点击 Finish，然后点击 OK

# 创建支持 C/C++ 的新项目
## 开发环境
- Mac 系统
- Android Studio 3.3.2
- Gradle 版本：gradle-4.10.1-all

## 搭建步骤
因为使用的是 Android Studio 3.3.2，在创建新项目的时候没有 ``Include C++ Support`` 这个选项了，所以现在先创建基础项目，再关联C/C++
1. 创建基础项目
2. 创建新的原生源文件，并添加到项目中
3. 创建 CMake 构建脚本，将原生源代码构建到库中
4. Gradl 与原生库关联

### 创建新的原生源文件
在应用模块的主源代码集中创建一个包含新建原生源文件的``cpp/``目录，步骤如下：
1. 选择 Project 视图
2. 导航到 ``自己模块 > src > main`` ,在 `main` 目录中创建一个新的文件夹`cpp`
3. 在``cpp``目录中创建``C/C++``文件；创建此文件时可以选择创建标头文件

### 创建 CMake 构建脚本
CMake 构建脚本是一个纯文本文件，您必须将其命名为 `CMakeLists.txt`。这里只介绍了基本命令
1. 选择 Project 视图
2. 在自己模块的根目录下创建文件，命名`CMakeLists.txt`
3. 配置脚本
```
# 设置 CMake 的最低版本
cmake_minimum_required(VERSION 3.4.1)

# Specifies a library name, specifies whether the library is STATIC or
# SHARED, and provides relative paths to the source code. You can
# define multiple libraries by adding multiple add.library() commands,
# and CMake builds them for you. When you build your app, Gradle
# automatically packages shared libraries with your APK.

add_library( # Specifies the name of the library. 这个库的名字
        main

        # Sets the library as a shared library. 这个库的类型
        SHARED

        # Provides a relative path to your source file(s). 源文件的路径
        src/main/cpp/main.cpp)

# Specifies a path to native header files. 标头文件的路径
include_directories(src/main/cpp/include/)

```
4. 最后生成的原生库，是有命名规范的
```
lib库名称.so
```

### Gradle 与原生库关联
要将 Gradle 关联到您的原生库，您需要提供一个指向 CMake 或 ndk-build 脚本文件的路径。在您构建应用时，Gradle 会以依赖项的形式运行 CMake 或 ndk-build，并将共享的库打包到您的 APK 中。Gradle 还使用构建脚本来了解要将哪些文件添加到您的 Android Studio 项目中，以便您可以从 Project 窗口访问这些文件。如果您的原生源文件没有构建脚本，则需要先创建 CMake 构建脚本，然后再继续。

将 Gradle 关联到原生项目后，Android Studio 会更新 Project 窗格以在 cpp 组中显示您的源文件和原生库，在 External Build Files 组中显示您的外部构建脚本。

注：更改 Gradle 配置时，请确保通过点击工具栏中的 Sync Project  应用更改。此外，如果在将 CMake 或 ndk-build 脚本文件关联到 Gradle 后再对其进行更改，您应当从菜单栏中选择 Build > Refresh Linked C++ Projects，将 Android Studio 与您的更改同步。

关联步骤：
1. 从 IDE 左侧打开 Project 窗格并选择 Android 视图
2. 右键点击您想要关联到原生库的模块（例如 app 模块），并从菜单中选择 Link C++ Project with Gradle
3. 选择 CMake，请使用 Project Path 旁的字段为您的外部 CMake 项目指定 CMakeLists.txt 脚本文件
4. 点击OK就可以了

## 检测是否成功
所有工作完成之后，打包APK，对APK进行分析，发现在`lib`目录下有CPU架构文件夹，每一个文件夹中都有`libmain.so`文件，就说明成功了。
步骤如下：
1. Build > Build Bundle(s)/ APK(S) > Buidl APK(S)
2. 等待打包完成，点开`Anaylze APK`
3. 出现的窗体中，打开`lib`目录，打开其中一个CPU架构目录(例如x86)，就会看见`libmain.so`

# 利用 JNI 调用原生库
JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信（主要是C&C++）
## 使用方法
以上面的原生库为例子，介绍 JNI
### 源文件 main.cpp
```
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring

JNICALL
Java_com_hongliang_ndkdemo_NDKMainHelper_getNDKPrint(JNIEnv *env, jobject /* this */) {
    std::string hello = "NDKDemo";
    return env->NewStringUTF(hello.c_str());
}

```

- ``#include <jni.h>`` 声明此文件要使用 jni 的内容
- ``#include <string>`` 因为需要使用到 String，所以要声明
- ``Java_com_hongliang_ndkdemo_NDKMainHelper_hello`` 这个是最重要的，因为是有命名规范的：``Java_包名_类名_方法名``。可以理解为调用com.hongliang.ndkdemo包下NDKMainHelper类的hello方法，实际是调用 main.cpp 的 ``Java_com_hongliang_ndkdemo_NDKMainHelper_hello`` 函数

### Java 文件 NDKMainHelper
```
package com.hongliang.ndkdemo;
public class NDKMainHelper {

    static {
        System.loadLibrary("main");
    }

    public native String getNDKPrint();
}
```
- static 静态块中加载原生库 ``System.loadLibrary("main")``，使用的是库名
- native 标志此方法是调用原生库的函数

### Activity文件
```
package com.hongliang.ndkdemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        NDKMainHelper ndkMainHelper = new NDKMainHelper();
        String msg = ndkMainHelper.getNDKPrint();
        Log.e("MainActivity", msg);
    }
}

```
- 在 MainActivity 中创建 NDKMainHelper 的对象，并调用 getNDKPrint 方法，此时就会调用 main.cpp 的函数了。
- 最终打印出来的是：NDKDemo

# 文献
[官方文档](https://developer.android.google.cn/studio/projects/add-native-code)
