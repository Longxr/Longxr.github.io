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
Qt-mingw工程、Qt-msvc工程、VS工程相互转换，包括子工程动态库、第三方库设置，windows下就应该老老实实用msvc，用mingw的话win8以上的新特性都没得玩<!-- more -->
<The rest of contents | 余下全文>

### Qt-mingw工程转换为Qt-msvc工程
> VS不太会用，先在QtCreator调好了再扔VS..._(:з)∠)_

#### 编译错误处理
- 第三方库更换，将.a库都换成.lib库
- 动态库没有导出lib文件，在mingw不在导出类上写Q_DECL_EXPORT也能导出相关的.a，VS不写的话不会导出.lib
- 用到windows.h或者网络相关就会有一个蛋疼的windows.h、winsock.h、winsock2.h各种重定义的问题，在使用windows.h之前：
```
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
```
- 其他各种函数符号未解析什么的，把函数名用必应搜一下，就会跳到windows API文档，在pro中加上相关lib
```
LIBS += \
    -ldwmapi -lOle32 -lwinmm -lksuser -luuid -lUser32
```

- Qt5mingw中的connect函数接收者为this时，可以不写槽函数的作用域，VS中不写会报错

#### 编译警告处理
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
将对应的文件字符编码修改为utf-8 BOM

### Qt工程转换为VS工程

1. 打开Qt的命令行工具`Qt 5.10.1 32-bit for Desktop (MSVC 2015)`，先到子工程目录输入`qmake -tp vc XXX.pro`,生成对应的vcxproj
2. 到总工程目录输入`qmake –tp vc`,生成对应的解决方案sln

3. 翻译文件如果放在资源文件中，会被加到资源文件过滤器和翻译文件过滤器，可能会在vsproject的配置文件中重复，删除掉重复的翻译文件`<ItemGroup>...xxx.ts...</ItemGroup>`

### VS工程转换为Qt工程

1. VS装上Qt VS Tool，点击插件的Create basic .pro file

2. 修改生成的.pro文件：
```
TEMPLATE = lib
Release:DESTDIR = xxx/Release
Debug:DESTDIR = xxx/Debug
INCLUDEPATH += ... //添加第三方头文件引用路径
LIBS += ....//添加第三发库文件
```

3. 双击改好的.pro文件，QT Creator打开此工程，选中合适的Kit，就可以build了。看你之前相关库有没有限制，我之前是32程序就选的QT 5.10.1的msvc2015。

4. 要运行VS工程的时候记得把Qt-msvc的bin目录加到环境变量Path里`C:\Qt\Qt5.10.1\5.10.1\msvc2015\bin`。

5. 编译64位的程序可能会有个坑，虽然我没碰到但是还是记录下.
在系统环境变量%PATH%里，对于Visual Studio的编译器cl.exe和链接器link.exe, 要选对路径。比如，对于64位的机器，路径 `C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64` 应该被加到%PATH%中，而不是` C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin`。

### VS2013工程转换为VS2015工程
这个最好是重新添加文件搞一个，直接点VS的那个升级会找不到Qt的相关头文件，会报一排错......
