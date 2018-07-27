---
title: Qt-Mingw-ffmpeg
categories:
  - Qt
tags:
  - Qt
  - ffmpeg
date: 2018-07-27 08:38:53
---

<Excerpt in index | 摘要> 
介绍下在Qt下直接调用官方下载的动态库以及自己编译成静态库等，另外关于ffmpeg怎么用算LGPL谁知道的话求告知<!-- more -->
<The rest of contents | 余下全文>


### 直接使用动态库

#### 官网下载
如果没有特殊要求，可以直接在ffmpeg[官网下载](https://ffmpeg.zeranoe.com/builds/)编译好的库文件直接使用，自己编译还是相当麻烦的QAQ。选择自己操作系统相关的版本，下载shared和dev两个压缩包：
![官网下载](https://upload-images.jianshu.io/upload_images/2756183-de294d5649275595.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Static包含的是静态编译的ffmpeg.exe，Shared包含的是ffmpeg的动态库及ffmpeg.exe,Dev中包含的是加入到工程中的lib（使用的时候还是要添加动态库才能运行）。

#### 添加到工程

在pro文件中加上对应的libs，添加lib的时候不用写后缀名字：
```
-L$$SDKs/ffmpeg/lib -L$$SDKs/ffmpeg/bin -lavutil -lavformat -lavcodec -lswscale -lavdevice -lswresample 
```

在要使用的文件中引用ffmpeg相关的头文件：
![引用头文件](https://upload-images.jianshu.io/upload_images/2756183-7219099117b5dee4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 可能的问题
- 最新的ffmpeg有把libfmx (使用intel的qsv硬编硬解，下面有说明) 编译进去，但是好像没有fmx的头文件，会报错。可以去[这里](https://github.com/lu-zero/mfx_dispatch/tree/master/mfx)下下来，放到ffmpeg的include目录下。
- 编译没问题记得要把dev中的dll拷贝到生成的exe目录下，否则运行不起来。
- 相关工程可以参考[gayhub](https://github.com/Longxr/ffmpegtest)的demo。

### 自定义编译
官网下载的ffmpeg动态库编译的configure里有一条是`--enable gpl`，我们用的时候是直接用的动态库，我也不确定这算不算LGPL，实在没辙的话只能自己编译了。
![license相关](https://upload-images.jianshu.io/upload_images/2756183-87c6a43dc855ba41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### IDE安装
本人安装的是Qt5.7.0的mingw32bit，[下载地址戳我](http://download.qt.io/archive/qt/5.7/5.7.0/)

### msys2安装
因为要编译32bit的，下载选择msys2的32位版本，[下载地址戳我](http://www.msys2.org/)
安装完成后找到msys2安装目录下的msys2_shell.cmd，将其中一行的注释去掉：
![msys2_shell.cmd修改](https://upload-images.jianshu.io/upload_images/2756183-b5e7ed4fe9651426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开Qt的命令行工具，将目录切换到msys2的安装目录下，输入：`msys2_shell.cmd -mingw32`
![命令行输入](https://upload-images.jianshu.io/upload_images/2756183-9c2a7ab457c96085.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

msys2就打开了，下一步是安装相关的软件包。

### 软件包安装
在msys2中的包管理工具是pacman，安装命令如下：
```
pacman -S make
pacman -S pkg-config
pacman -S nasm
pacman -S git
pacman -S automake autoconf perl libtool
```

### 软编(编译libx264，这货是GPL的哈)
先下载源码，`git clone http://git.videolan.org/git/x264.git`,切换到x264仓库目录下：
```
./configure \
 --prefix=/usr/local \
 --enable-shared \
 --disable-cli
 (loading...)

 ./make -j$(nproc)
 (loading...)

make install
 (loading...)
```

### 硬编intel qsv准备
需要libmfx库 ，下载源码，`git clone https://github.com/lu-zero/mfx_dispatch.git`,切换到对应目录下：
```
autoreconf -i
 (loading...)

./configure --prefix=/usr/local
 (loading...)

make -j$(nproc)
 (loading...)

make install
 (loading...)
```

### 硬编Nvidia nvenc准备
添加nvidia codec头文件，`git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git`,切换到目录下：
```
make
make install
```
NVidia Video Codec SDK下载，[下载地址戳我](https://developer.nvidia.com/nvidia-video-codec-sdk#Download), 解压后把里面的NvCodec下的库文件拷贝到/usr/local/lib、头文件拷贝到/usr/local/include下

### 编译ffmpeg
下载ffmpeg源码，`git clone https://git.ffmpeg.org/ffmpeg.git`,切换到对应目录下：
```
PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
./configure \
--prefix=/usr/local/ffmpeg \
--disable-static \
--enable-shared \
--enable-gpl \
--target-os=mingw32 \
--enable-libx264 \
--enable-libmfx \
--enable-encoder=h264_qsv \
--enable-decoder=h264_qsv \
--enable-nvenc \
--enable-cuda \
--enable-cuvid \
--enable-nonfree \
--extra-cflags=-I/usr/local/include \
--extra-ldflags=-L/usr/local/lib
 (loading...)

make -j$(nproc)
 (loading...)

make install
 (loading...)
```

编译好后的ffmpeg在/usr/local/ffmpeg，就是configure第一行指定的位置。
最后贴下各种文件的目录：
![源码下载路径](https://upload-images.jianshu.io/upload_images/2756183-b897b5003b4c4377.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ffmpeg添加的lib路径](https://upload-images.jianshu.io/upload_images/2756183-69ae2370ca4b5e0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ffmpeg 添加的include路径](https://upload-images.jianshu.io/upload_images/2756183-bdb6f397407b4729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 可能的问题
* 测试编译，没有把--enable gpl去掉，不确定去掉能不能编译哈，毕竟....x264是GPL的。
* 编译好后的ffmpeg可以在Qt下正常使用qsv、nvenc编码，解码暂时没用到不确定。
* 编译出来的ffmpeg.exe没法双击运行，提示无法定位程序输入点__gxx_personality_v0于动态链接库avcodec-58.dll上。
* Nvidia的cuda硬解在mingw下貌似搞不了......[相关问题](https://stackoverflow.com/questions/5888908/cuda-with-mingw-updated)
