---
title: Qt5.12.0 + VS2017 环境搭建
categories:
  - Qt
tags:
  - VS2017
  - Qt5.12.0
  - Mingw32
date: 2018-12-30 13:49:51
---

<Excerpt in index | 摘要> 
Qt5.12.0在2018年12月5日发布，虽然平时工作还是在用Qt5.7.0的mingw32版(支持XP的最后一个mingw版本？)，放假了折腾下新版本看看 <!-- more -->
<The rest of contents | 余下全文>

## 构建版本选择
Qt5.9以上的Qt安装程序可以在安装的时候选择mingw版本和VS版本等，但是每个版本对mingw和VS、32位和64位的支持都有点不太一样.....项目要是迁移的话需要注意下，比如：

![Qt5.11.2](https://upload-images.jianshu.io/upload_images/2756183-aaae639a4c386d01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Qt5.12.0](https://upload-images.jianshu.io/upload_images/2756183-3830d98d92e1e861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Qt5.11.2有mingw32版，VS2015有32位和64位， VS2017只有64位；
* Qt5.12.0只有mingw64版，VS2017有32位和64位，VS2015只有64位。

首先，mingw32我是需要的，这个官方版本没提供，需要自行编译；VS的32位也是需要的，一是要用VS调试的话32位好迁移，二是VS对XP支持可以在安装时勾选 (5.7.0后mingw版貌似没辙了)。所以，安装环境就确定为Qt5.12.0 + VS2017 + 自行编译的mingw32部分。

## VS2017安装
VS2015还有iso安装的，2017好像只能在线安装了。VS2017安装器下载地址[戳这里](https://visualstudio.microsoft.com/zh-hans/downloads/)。Visual Studio Installer我开始装的时候点到VS2017 build Tool了，注意不是这个...我装的是2017的社区版，就上面第一个。
![VS2017社区版选择](https://upload-images.jianshu.io/upload_images/2756183-f31f9510dd2aee1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![VS2017安装组件](https://upload-images.jianshu.io/upload_images/2756183-ebe876e860f93fe1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

C++桌面开发是要勾选的，右边对XP的支持看你需要选择，另外其他组件根据自己需要选择，全家桶好像有50多个G，越来越大了。

## Qt5.12.0 安装
5.12.0下载地址[戳这里](https://download.qt.io/archive/qt/5.12/5.12.0/)，安装就组件那里根据需要装就行了，我是勾选的MSVC2017的32位、64位、还有mingw的64位。只要装好了VS2017，在Qt的构建套件里，相关的编译器、调试器就会自动识别了。要是调试器没有(装VS没勾win10 SDK啥的)，可以另外装[戳这里](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/debugger/debugger-download-tools)。

![Qt构建套件](https://upload-images.jianshu.io/upload_images/2756183-0a41d6ae339d8505.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## mingw32位安装
可以自己下载Qt的源码编译，正好看到网上有人放了个编译好的版本，[原下载地址](https://yadi.sk/d/oaPalrciE_lUJQ)。国外的网盘下载速度较慢，另外传到百度云了，[提取码：enf5](https://pan.baidu.com/s/18qURVWIL5gDEKyM1nqGbug) (其实百度云也挺慢的...)。下载解压后，复制到Qt安装目录，在构建套件中，新建一个mingw32的构建套件:

1. Debuggers：添加`C:\Qt\Qt 5.12.0\Tools\mingw730_32\bin\gdb.exe`
2. 编译器：添加C：`C:\Qt\Qt 5.12.0\Tools\mingw730_32\bin\gcc.exe`；添加C++：`C:\Qt\Qt 5.12.0\Tools\mingw730_32\bin\g++.exe`
3. Qt Version：添加`C:\Qt\Qt 5.12.0\5.12.0\mingw73_32\bin\qmake.exe`
4. 构建套件：添加一个新的mingw32套件，指定前面添加的编译器、调试器、Qt Versions，如图所示：

![mingw32构建套件](https://upload-images.jianshu.io/upload_images/2756183-d3a7dfd3624270c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另外，这个编译的mingw32有个小bug，找不到Qt的plugin目录，需要在mingw32构建套件的环境变量中加下Qt的插件目录，然后在Qt Creator这边编译生成32位的程序已经没有问题了。
![添加环境变量](https://upload-images.jianshu.io/upload_images/2756183-9357e69b583d468d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## VS Qt插件安装
一般我是在Qt Creator开发，VS暂时是没怎么用到，这里简单介绍下怎么在VS建立Qt工程。
1. 安装Qt的扩展：点击工具->扩展和更新->联机->搜索Qt，下载安装后重启即可。

![扩展和更新](https://upload-images.jianshu.io/upload_images/2756183-a5365cbb1b8f4784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![下载Qt扩展](https://upload-images.jianshu.io/upload_images/2756183-6e0061f5c2aab8ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.配置Qt MSVC目录，安装好后菜单会有一个Qt VS Tools，点击Qt Options，将32位和64位目录设置好。
32位：`C:\Qt\Qt5.12.0\5.12.0\msvc2017`，64位：`C:\Qt\Qt5.12.0\5.12.0\msvc2017_64`。

![配置Option](https://upload-images.jianshu.io/upload_images/2756183-320152ceb57b6826.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![配置Option](https://upload-images.jianshu.io/upload_images/2756183-55f29e663783dacb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 新建个项目测试下，不知道为啥模板在测试里=-=

![新建项目](https://upload-images.jianshu.io/upload_images/2756183-b79f26c25542d822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![运行结果](https://upload-images.jianshu.io/upload_images/2756183-55a8a646fbfcbd2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Creator工程、VS工程互相转换
详细内容可以看之前的这篇->[Qt-mingw工程、Qt-msvc工程、VS工程相互转换](https://www.jianshu.com/p/d57108ddd9de)，其他环境搭建问题碰到再补充。

