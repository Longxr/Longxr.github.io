---
title: Qt5.12.4 Android环境配置
categories:
  - Qt
tags:
  - Qt
  - Android
date: 2019-07-25 16:18:39
---

<Excerpt in index | 摘要> 
Qt5.12.4 Android环境配置，从入门到入土_(:з)∠)_ <!-- more -->
<The rest of contents | 余下全文>

## Java JDK 配置
### Java JDK 下载
[JDK下载地址戳这里](http://www.oracle.com/technetwork/java/javase/downloads/)

![JDK下载](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_01.png)

### 环境变量配置
安装过程略，一路下一步就行，安装完毕需要把Java相关路径添加到环境变量。
在环境变量里新建两个用户变量（用户变量是当前用户环境变量，系统变量是所有用户的）：
* JAVA_HOME，值为安装的Java JDK 目录，`C:\Program Files\Java\jdk1.8.0_221`
![JAVA_HOME](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_02.png)

* CLASSPATH，值为`.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`
![CLASSPATH](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_03.png)

再在Path里添加两个环境变量：
* `%JAVA_HOME%\bin`
* `%JAVA_HOME%\jre\bin`
![环境变量](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_04.png)

设置完点确定就可以在命令行输入`java -version`，来测试下环境变量设置好没。
![java -version](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_05.png)

## NDK 配置
[NDK 下载地址戳这里](https://developer.android.google.cn/ndk/downloads/)
下载特定的版本r19c，`https://dl.google.com/android/repository/android-ndk-r19c-windows-x86_64.zip`
下载后直接解压到一个路径不包含空格的目录即可，包含空格Qt不识别。
![NDK解压](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_06.png)

## Android SDK 配置
### 仅安装SDK （具体参考下方其他博客链接）
下载地址：`https://dl.google.com/android/repository/sdk-tools-windows-4333796.zip`。
同上NDK，解压即可。参考位置 `C:\Android\sdk-tools-windows-4333796`。
如果您不需要 Android Studio，可以从下面下载基本的 Android 命令行工具。您可以使用随附的 sdkmanager 下载其他 SDK 软件包。
[sdkmanager | Android Developers](https://developer.android.com/studio/command-line/sdkmanager)。
命令行执行`sdkmanager "build-tools" "platform-tools" "platforms;android-28"`。

### 安装Android Studio
[下载地址戳这里](https://www.androiddevtools.cn/)
我是之前已经装好了Android Studio了，就直接用其自带的SDK Manager了。相关配置仅供参考，我也不确定哪些是必须的哈，勾选9.0因为我手机就是Android9.0的。
![SDK Platforms](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_07.png)
![SDK Tools](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_08.png)

***Android 相关的下载大部分都需要翻墙，最省事的是将小飞机的代理模式改为全局模式***
![小飞机全局模式](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_09.png)

## Qt配置
  [Qt各版本下载戳这里](https://download.qt.io/official_releases/qt/)，我这里用的是5.12.4，此版本有个重大更新：logo换了233333
![5.12.2](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_10.png)
![5.12.4](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_11.png)

Qt5.12.4下载链接，`http://iso.mirrors.ustc.edu.cn/qtproject/archive/qt/5.12/5.12.4/qt-opensource-windows-x86-5.12.4.exe`
构建套件Android的3个都勾上了，基本不占多少空间，这里吐槽下Mingw一个就要5GB多...
![构建套件选择](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_12.png)

安装好后打开Qt Creator，在选项->设备里，填好上面设置的JDK、Android SDK、NDK路径。
![配置Android相关路径](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_13.png)

## 运行Demo测试
### 新建项目
Qt新建个项目，点那个像手机的，构建套件把Android的都勾上，方便切换不勾选这么多也行。
![新建项目](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_14.png)

![构建套件选择](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_15.png)

### 开启USB调试
我这里直接真机调试，手机要允许USB调试，小米6开启是在设置->更多设置->开发者选项->USB调试、USB安装，这两个选项打开。
![开启USB调试](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_16.jpeg)

### 编译运行遇到的问题
点击Qt Creator的绿色小三角就开始编译运行了，此时手机会提示允许USB调试，允许后就能看到自己的手机了，手机允许后此界面没刷新可以点左下角的`Refresh Device List`。
![手机设备](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_17.png)

编译过程很慢，可以看到查看下方编译输出，提示为`Downloading https://services.gradle.org/distributions/gradle-4.6-bin.zip`，这个很迷，开着小飞机也下不动，但是复制下载链接到浏览器立马就能下下来。
![编译输出提示](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_18.png)

取消编译，将下载好的gradle-4.6-bin.zip拷贝到下载目录，下载的两个临时文件gradle-4.6-bin.zip.lck、gradle-4.6-bin.zip.part删掉。
![gradle-4.6-bin.zip下载](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_19.png)

重新开始编译，后面就很顺利，可以直接运行了。
![滚动Demo运行](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Android-environment_20.jpeg)

## 参考链接
* [QT--Android之全配置教程](https://blog.csdn.net/future_ai/article/details/85616180)
* [Qt-For-Android配置](https://zx9229.github.io/2019/01/26/Qt-For-Android%E9%85%8D%E7%BD%AE/)


