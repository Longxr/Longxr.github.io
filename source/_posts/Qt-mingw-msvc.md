---
title: Qt-mingw工程、Qt-msvc工程、VS工程相互转换
categories:
  - Qt
tags:
  - mingw
  - msvc
  - pro
date: 2018-08-22 13:34:26
---

<Excerpt in index | 摘要>
Qt-mingw 工程、Qt-msvc 工程、VS 工程相互转换，包括子工程动态库、第三方库设置，windows 下就应该老老实实用 msvc，用 mingw 的话 win8 以上的新特性都没得玩<!-- more -->
<The rest of contents | 余下全文>

## Qt-mingw 工程转换为 Qt-msvc 工程

> VS 不太会用，先在 QtCreator 调好了再扔 VS..._(:з)∠)_

### 编译错误处理

- 第三方库更换，将.a 库都换成.lib 库
- 动态库没有导出 lib 文件，在 mingw 不在导出类上写 Q_DECL_EXPORT 也能导出相关的.a，VS 不写的话不会导出.lib
- 用到 windows.h 或者网络相关就会有一个蛋疼的 windows.h、winsock.h、winsock2.h 各种重定义的问题，在使用 windows.h 之前：

```
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
```

- 其他各种函数符号未解析什么的，把函数名用必应搜一下，就会跳到 windows API 文档，在 pro 中加上相关 lib

```
LIBS += \
    -ldwmapi -lOle32 -lwinmm -lksuser -luuid -lUser32
```

- Qt5mingw 中的 connect 函数接收者为 this 时，可以不写槽函数的作用域，VS 中不写会报错

### 编译警告处理

- warning: LNK4098: 默认库“LIBCMT”与其他库的使用冲突；请使用 /NODEFAULTLIB:library。
  修改`C:\Qt\Qt5.10.1\5.10.1\msvc2015\mkspecs\common`下的**msvc-desktop.conf**：

```
//修改前
QMAKE_LFLAGS_DEBUG      = /DEBUG

//修改后
QMAKE_LFLAGS_DEBUG      = /DEBUG /NODEFAULTLIB:libcmt.lib /NODEFAULTLIB:msvcrt.lib /NODEFAULTLIB:libcmtd.lib
```

- warning: 未引用的形参。
  修改`C:\Qt\Qt5.10.1\5.10.1\msvc2015\mkspecs\common`下的**msvc-desktop.conf**：

```
//修改前
QMAKE_CXXFLAGS_WARN_ON  = $$QMAKE_CFLAGS_WARN_ON -w34100 -w34189 -w44996

//修改后
QMAKE_CXXFLAGS_WARN_ON  = $$QMAKE_CFLAGS_WARN_ON -w34189 -w44996
```

- warning: C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
  将对应的文件字符编码修改为 utf-8 BOM

## Qt 工程转换为 VS 工程

### 通过扩展

1. 安装 Qt Visual Studio Tools 拓展，配置好 Qt options 中的 MSVC 路径
2. 点击工具栏 Qt VS Tools，Open Qt Project File，选择工程的.pro 文件后就能自动生成 sln 了

![Qt-mingw-msvc-2020-04-16-10-26-03](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-mingw-msvc-2020-04-16-10-26-03.png)

### 命令行

先搞个 bat 把指定编译环境加到环境变量，比如下面就是 Qt5.12.5 + msvc2017 构建套件 + vs2019 的环境：

```bat
SET DEPLOY_QT_VERSION=5.12.5
SET MSVC_VERSION=2019
SET DEPLOY_QT_COMPILE=msvc2017
SET PATH=%PATH%;C:\Qt\Qt%DEPLOY_QT_VERSION%\%DEPLOY_QT_VERSION%\%DEPLOY_QT_COMPILE%\bin;
SET PATH=%PATH%;C:\Qt\Qt%DEPLOY_QT_VERSION%\Tools\QtCreator\bin;
SET PATH=%PATH%;C:\Program Files (x86)\Microsoft Visual Studio\%MSVC_VERSION%\Community\VC\Auxiliary\Build;

call vcvarsall.bat amd64_x86
```

在最外层工程目录运行：`qmake -tp vc -r`。会递归生成子目录的 vcxproj，并生成总工程的 sln。

## VS 工程转换为 Qt 工程

1. VS 装上 Qt VS Tool，点击插件的 Create basic .pro file

2. 修改生成的.pro 文件：

```
TEMPLATE = lib
Release:DESTDIR = xxx/Release
Debug:DESTDIR = xxx/Debug
INCLUDEPATH += ... //添加第三方头文件引用路径
LIBS += ....//添加第三发库文件
```

3. 双击改好的.pro 文件，QT Creator 打开此工程，选中合适的 Kit，就可以 build 了。看你之前相关库有没有限制，我之前是 32 程序就选的 QT 5.10.1 的 msvc2015。

4. 要运行 VS 工程的时候记得把 Qt-msvc 的 bin 目录加到环境变量 Path 里`C:\Qt\Qt5.10.1\5.10.1\msvc2015\bin`。

5. 编译 64 位的程序可能会有个坑，虽然我没碰到但是还是记录下.
   在系统环境变量%PATH%里，对于 Visual Studio 的编译器 cl.exe 和链接器 link.exe, 要选对路径。比如，对于 64 位的机器，路径 `C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64` 应该被加到%PATH%中，而不是`C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin`。

## VS2013 工程转换为 VS2015 工程

这个最好是重新添加文件搞一个，直接点 VS 的那个升级会找不到 Qt 的相关头文件，会报一排错......
