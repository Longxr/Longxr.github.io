---
title: Qt5.12.5 静态编译以及裁剪大小对比
categories:
  - Qt
tags:
  - static build
date: 2020-07-11 18:49:35
---

<Excerpt in index | 摘要>
使用 VS2019 静态编译 Qt5.12.5 源码，并记录不同配置裁剪后生成的程序大小 <!-- more -->
<The rest of contents | 余下全文>

最近在研究用静态编译的 Qt 做个 Qt 的安装包程序，因为 nsis 在界面自定义还有多语言适配等问题上不太理想。做安装包程序基本上只需要有最基础的 Qt 界面模块即可，不过把除了 UI 的模块全部去掉又比较难用...这里记录下两种极端编译配置下生成的程序大小作为参考。

# 编译环境搭建

## 编译工具安装

编译需要的环境可以看官方文档，[windows-requirements](https://doc.qt.io/qt-5/windows-requirements.html) 中的`Building from Source`有说明需要安装的工具，所有工具都记得要添加到环境变量。

![Qt5-12-5-static-build-2020-07-11-22-37-34](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-11-22-37-34.png)

1. VS2019（其他版本仅作参考），下载地址：[Visual Studio 2019](https://visualstudio.microsoft.com/zh-hans/vs/)
2. ActivePerl， 下载地址：[Download And Install Perl: ActivePerl](https://www.activestate.com/products/perl/downloads/)
3. Python，下载地址：[Python Releases for Windows](https://www.python.org/downloads/windows/)
4. Ruby（以前的官方文档是有的，如果有问题可以装下），下载地址：[RubyInstaller for Windows](https://rubyinstaller.org/downloads/)
5. Jom，多核编译，下载地址：[jom releases](http://download.qt.io/official_releases/jom/jom.zip)

## 编译准备

1. 源码下载并解压，下载地址：[qt-everywhere-src-5.12.5.zip](https://download.qt.io/archive/qt/5.12/5.12.5/single/qt-everywhere-src-5.12.5.zip)
2. 在源码同级目录创建个 qt-build 目录来放生成的文件，也就是使用`shadow build`，方便修改配置后进行多配置编译。直接运行源码目录下的 configure.bat 会把生成文件生成到源码目录，修改配置后编译容易出错。

![Qt5-12-5-static-build-2020-07-11-22-47-19](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-11-22-47-19.png)

3. 修改`qtbase\mkspecs\common\msvc-desktop.conf`文件，将 MD 改为 MT

![Qt5-12-5-static-build-2020-07-12-00-27-17](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-27-17.png)

4. 打开 VS 的命令行工具并切换到第 2 步创建的 qt-build 目录，后面就是输入命令编译了。

![Qt5-12-5-static-build-2020-07-12-00-29-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-29-11.png)

# 编译步骤

```shell
.\..\src\configure.bat xxx  # 根据指定配置生成makefile
nmake 或者 jom              # 编译
nmake install               # 安装到指定目录
```

# 全模块配置

## 编译配置命令

最后的-mp 是多核编译的参数，多核编译用 jom 快得一匹，全模块也只要 20 分钟编译完成。

```bat
.\..\src\configure.bat -static -release -platform win32-msvc -release -static -prefix "D:\Qt\Qt5.12.5-msvc2019-x86-static-release-full" -opensource -confirm-license -nomake examples -nomake tests -plugin-sql-sqlite -plugin-sql-odbc -qt-zlib -qt-libpng -qt-libjpeg -opengl desktop -mp
```

## Qt库大小
![Qt5-12-5-static-build-2020-07-12-10-15-19](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-10-15-19.png)

## Demo 程序大小
![Qt5-12-5-static-build-2020-07-12-00-02-46](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-02-46.png)

# 超精简配置

这个配置把能删的模块都删了，编译出来后基本也就基础UI相关类能用...所以编译起来超快的。

## 编译配置命令

```bat
.\..\src\configure.bat -static -release -platform win32-msvc -no-opengl -prefix "D:\Qt\Qt5.12.5-msvc2019-x86-static-release-mini" -opensource -confirm-license -make libs -nomake tools -nomake examples -nomake tests -skip qt3d -skip qtandroidextras -skip qtcanvas3d -skip qtconnectivity -skip qtdatavis3d -skip qtdeclarative -skip qtdoc -skip qtgamepad -skip qtcharts -skip qtgraphicaleffects -skip qtimageformats -skip qtlocation -skip qtmacextras -skip qtmultimedia -skip qtnetworkauth -skip qtpurchasing -skip qtquickcontrols -skip qtquickcontrols2 -skip qtscript -skip qtscxml -skip qtsensors -skip qtserialbus -skip qtspeech -skip qtsvg -skip qttools -skip qttranslations -skip qtvirtualkeyboard -skip qtwayland -skip qtwebchannel -skip qtwebengine -skip qtwebsockets -skip qtwebview -skip qtwinextras -skip qtx11extras -skip qtxmlpatterns -no-feature-texthtmlparser -no-feature-textodfwriter -no-feature-concurrent -no-feature-effects -no-feature-sharedmemory -no-feature-systemsemaphore -no-feature-im -no-feature-dom -no-feature-filesystemwatcher -no-feature-graphicsview -no-feature-graphicseffect -no-feature-sizegrip -no-feature-printpreviewwidget -no-feature-keysequenceedit -no-feature-colordialog -no-feature-fontdialog -no-feature-printpreviewdialog -no-feature-progressdialog -no-feature-errormessage -no-feature-wizard -no-feature-datawidgetmapper -no-feature-cups -no-feature-paint_debug -no-feature-codecs -no-feature-big_codecs -no-feature-iconv -no-feature-networkproxy -no-feature-socks5 -no-feature-networkdiskcache -no-feature-bearermanagement -no-feature-mimetype -no-feature-undocommand -no-feature-undostack -no-feature-undogroup -no-feature-undoview -no-feature-statemachine -no-feature-gestures -no-feature-dbus -no-feature-sessionmanager -no-feature-topleveldomain -no-feature-sha3-fast -no-feature-imageformat_ppm -no-feature-imageformat_xbm -no-feature-freetype -no-feature-appstore-compliant -no-feature-process -no-feature-lcdnumber -qt-zlib -qt-libpng -qt-libjpeg
```

## Qt库大小
![Qt5-12-5-static-build-2020-07-12-10-16-54](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-10-16-54.png)

## Demo 程序大小

![Qt5-12-5-static-build-2020-07-12-00-05-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-05-11.png)

比全模块配置生成的exe减少了 30%的大小，不过 30%也就 3M，在 windows 平台的话倒是没必要追求这种极致了=-=

# 使用编译后的静态库

1. 新建一个 Qt Version，并添加编译后的 qmake.exe 的路径。

![Qt5-12-5-static-build-2020-07-12-00-12-39](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-12-39.png)

此时如果是移动了编译后的路径，可能报错*Qt version is not properly installed,please run make install*。在 qmake.exe 所在目录新建一个 qt.conf 文件，并添加下面的内容即可：

```conf
[paths]
Prefix = ..
```

2. 新建一个构建套件，选好编译器和上一步新建的 Qt version 即可：

![Qt5-12-5-static-build-2020-07-12-00-24-17](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt5-12-5-static-build-2020-07-12-00-24-17.png)

# 参考链接

- [QtLite 使用方法，以及缩减应用体积的效果](https://blog.csdn.net/wsj18808050/article/details/55808104)
- [Qt 最新版 5.12 在 Windows 环境静态编译安装和部署的完整过程(VS2017)](https://blog.csdn.net/zhangpeterx/article/details/86529231)
