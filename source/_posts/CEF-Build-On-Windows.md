---
title: windows平台编译CEF支持MP3、MP4
categories:
  - CEF
tags:
  - windows
  - CEF
date: 2021-07-24 16:27:21
---

<Excerpt in index | 摘要>
CEF默认不支持mp3、mp4，需要的话得自己编译，记录在window下编译过程中遇到的各种坑 <!-- more -->
<The rest of contents | 余下全文>

## CEF基础版
如果不需要mp3、mp4的支持，可以直接下载构建好的版本，下载链接[戳这里](https://cef-builds.spotifycdn.com/index.html)。
新版本是cef和chromium在一个压缩包里，老版本是分开的。

![CEF-Build-On-Windows-2021-07-24-22-31-09](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/CEF-Build-On-Windows-2021-07-24-22-31-09.png)

## 准备工作
google的项目，都要配置好代理，想直接下是不行的。找个路径放相关的文件，比如`E:\cef`。

### 时区设置
设置系统区域为英语（美国）。（控制面板-区域-管理-更改系统区域设置-英语（美国）），设置完需要重启。防止编译一些字符集的报错。

### 代理设置
不知道是啥的话后面也不用搞了，不过就算设置了也还是慢，最好还是云服务器编译。

#### V2ray代理设置

1. 路由设置->勾选启用高级功能
2. 右下角小图标选择全局

#### git clone 代理配置
```bash
//加大buffer
git config --global http.postBuffer 524288000

//设置代理，ssr通常是1080，v2ray是10809，客户端底部有显示
git config --global http.proxy http://127.0.0.1:10809 && git config --global https.proxy https://127.0.0.1:10809

//取消代理（上面端口设置错了的话）
git config --global --unset http.proxy && git config --global --unset https.proxy
```

#### windows设置CMD代理
```bash
netsh
netsh>winhttp
netsh winhttp>
netsh winhttp>set proxy 127.0.0.1:10809      //设置代理
netsh winhttp>show proxy                     //查看代理
netsh winhttp>reset proxy                    //重置代理，下载完了不想要代理才用
```

#### 设置Boto代理
```
[Boto]
proxy = 127.0.0.1
proxy_port = 10809
```
创建.boto文件，在命令行设置环境变量，`set NO_AUTH_BOTO_CONFIG=E:\cef\.boto`

### 构建脚本
构建脚本会执行下载和编译，通过指定参数达到想要的编译效果，[下载链接](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py)。下载好后放到`E:\cef`目录下。这是个python脚本，python也需要装下，编译旧版本需要安装python2，新版本需要安装python3，在执行脚本的时候环境不对会给出提示。

#### 参数说明
automate-git.py 的参数可以手动执行 python automate-git.py --help 来查看，下面列举大部分情况都会用的。

* --branch 表示你要下载和编译哪个版本的代码，Chromium和CEF的版本对应关系[可以看这个链接](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-automated-method)
* --no-build 表示只下载代码而不编译
* --no-distrib 不执行打包项目，这里只为下载代码，我们还要修改支持多媒体的参数，所以不进行打包
* --force-clean 如果你曾经执行过这个脚本，可能会出错，则加上这个参数，它执行清理残留文件（你也可以手动在 chromium 源码目录执行 git clean -xdf 来清理目录中的多余内容）。
* --download-dir 下载的源码目录
* --checkout 如果你不想编译某个分支的最新版代码，可以指定具体的提交，只用设置CEF的提交就行，Chromium会跟着切换
* --force-clean-deps 编译老版本的时候会碰到下载第三方依赖库失败，碰到了可以加下这个
* --no-depot-tools-update 如果工具包之前下载过了可以加上，就不升级工具包了，第一次下载不加
* --no-update 编译时用，不更新直接编译
* --no-debug-build 编译时用，不编译debug只要release
* --build-log-file 生成编译的日志文件，出错可以看下
* --x64-build 构建64位的版本 


准备好后目录如下：
![CEF-Build-On-Windows-2021-07-24-22-53-01](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/CEF-Build-On-Windows-2021-07-24-22-53-01.png)

## CEF源码下载

下载前需要设置下环境变量，也可以直接在命令行输入就行。

```bat
set CEF_USE_GN=1
set GN_DEFINES=is_official_build=true
set GYP_DEFINES=buildtype=Official
set GYP_MSVS_VERSION=2017
set CEF_ARCHIVE_FORMAT=tar.bz2

@REM 第一次源码下载不加--no-depot-tools-update，会先包工具包下下来
python automate-git.py --download-dir=e:\cef\source --branch=3626 --checkout=90eb8cc --no-build --no-distrib --force-clean --force-clean-deps --no-depot-tools-update 
```

**有一个注意点，depot-tools内部有一些bat和python脚本，需要把它的路径设置到环境变量的PATH里，不然下载会报错**

### 下载错误处理
#### python缺库
![CEF-Build-On-Windows-2021-07-24-23-19-10](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/CEF-Build-On-Windows-2021-07-24-23-19-10.png)
看到脚本自己去下了python库的.whl，但是安装没成功，我就手动装了下：
```bash
pip install numpy=1.12.1
pip install opencv_python=3.2.0.7
pip install psutil=5.2.2
```

### No module named win32file
Traceback (most recent call last):
  File "../../build/toolchain/win/tool_wrapper.py", line 29, in <module>
    import win32file    # pylint: disable=import-error
ImportError: No module named win32file

修改方法：在错误日志里指定的目录安装pywin32，[参考链接](https://magpcss.org/ceforum/viewtopic.php?f=6&t=17258)

```bash
cd E:\cef\source\depot_tools\bootstrap-2@3_8_10_chromium_20_bin\python\bin
pip -m install pywin32
```


#### 提示subprocesss返回值非0
![CEF-Build-On-Windows-2021-07-24-23-23-12](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/CEF-Build-On-Windows-2021-07-24-23-23-12.png)

把depot-tools的路径设置到环境变量的PATH里:
```bat
set PATH=E:\cef\source\depot_tools;%PATH%;
```

#### AttributeError: 'module' object has no attribute 'CheckCallAndFilterAndHeader'

老版本编译才有的问题，需要修改代码，`E:\cef\source\cef\tools\gclient_util.py`，[参考链接](https://bitbucket.org/chromiumembedded/cef/issues/2736/gclient_hookpy-fails-because-of-missing)

命令行出问题的话，脚本就给个提示返回值非0，没把错误打印出来就很坑...把显示的命令行在终端直接敲一下就能看到具体错误了。
踩完这一路的坑，就可以开始编译了...

### CEF源码编译
编译需要设置的参数最重要的就是添加MP3、MP4支持 使用此条指令，不加白折腾了。还有就是指定VS的路径，编译报错了提示少什么路径可以再加。

```bat
set CEF_USE_GN=1
set GN_DEFINES=is_official_build=true
set GYP_DEFINES=buildtype=Official
set GYP_MSVS_VERSION=2017
set CEF_ARCHIVE_FORMAT=tar.bz2
@REM 添加MP3、MP4支持 使用此条指令
set GN_DEFINES=is_official_build=true proprietary_codecs=true ffmpeg_branding=Chrome
 
set GYP_GENERATORS=ninja,msvs-ninja
set GN_ARGUMENTS=--ide=vs2017 --sln=cef --filters=//cef/*
@REM VS2017安装在默认目录，但任然需要下面设置，可能是由于VS2015和VS2017同时安装，路径根据自己的安装目录和版本确定
set WIN_CUSTOM_TOOLCHAIN=1
set CEF_VCVARS=none
set GYP_MSVS_OVERRIDE_PATH=C:\Program Files (x86)\Microsoft Visual Studio\2017\Community
set SDK_ROOT=C:\Program Files (x86)\Windows Kits\10

@REM 编译64位调用VS的脚本有问题，环境变量还是去找的32位路径，下面改手动设置路径
@REM call "C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Auxiliary/Build/vcvarsall.bat" x64

@REM set INCLUDE=C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\shared;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\include;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\atlmfc\include;%INCLUDE%
@REM set PATH=C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\bin\HostX64\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\bin\HostX64\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Redist\MSVC\14.16.27012\x64\Microsoft.VC141.CRT;%PATH%
@REM set LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.17763.0\um\x86;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.17763.0\ucrt\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\lib\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\atlmfc\lib\x86;%LIB%
@REM set VS_CRT_ROOT=C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\crt

set INCLUDE=C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.17763.0\shared;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\include;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\atlmfc\include;%INCLUDE%
set LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.17763.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.17763.0\ucrt\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\lib\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\atlmfc\lib\x64;%LIB%
set VS_CRT_ROOT=C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\crt
set PATH=C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\bin\HostX64\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\bin\HostX64\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Redist\MSVC\14.16.27012\x64\Microsoft.VC141.CRT;%PATH%

python automate-git.py --download-dir=e:\cef\source --branch=3626 --checkout=90eb8cc --no-update --no-debug-build --build-log-file --verbose-build --force-distrib --force-build --x64-build
```

### 输出文件
在目录`E:\cef\source\chromium\src\cef\binary_distrib`下，还把压缩包也生成了。
生成的文件里没有libcef_dll_wrapper.lib，需要的话拿cmake生成一个vs工程再编译就行，什么配置都不用改，就略过了。

![CEF-Build-On-Windows-2021-07-25-00-22-37](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/CEF-Build-On-Windows-2021-07-25-00-22-37.png)


## 编后感
下载和编译环节一定要把参数都搞好，下载搞了2天，编译也搞了2天，心累...一开始直接编译了一个32位的，后面才反应过来要编译一个64位的，编译的时候CPU直接100%，其他活都不用干了。
等搞完发现指定分支提交和项目中使用的版本还不一样，又要重新checkout再编译Orz
最后发现还是搞个windows云服务器省事，环境直接选美国，代理也不用设置，我本地代理也只能3M/S，服务器下载30M/S，服务器编译还不影响本地工作(￣▽￣)"


## 参考链接：

- [官方构建Wiki](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md)
- [CEF版本号对应关系](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-automated-method)
- [在Windows上编译 CEF3 且加入mp3/mp4的支持](https://blog.csdn.net/baidu_32237719/article/details/79509819)
- [CEF学习（一）：下载和构建chromium，增加mp4等多媒体支持](https://blog.csdn.net/pnhuangyu/article/details/103191466/)


