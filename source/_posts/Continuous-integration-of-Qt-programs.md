---
title: Qt程序的持续集成
categories:
  - Qt
tags:
  - Qt
  - Continuous integration
  - mingw
  - msvc
date: 2019-03-02 17:00:17
---

<Excerpt in index | 摘要> 
持续集成用到的相关软件TeamCity，这里就不多介绍了，主要说明下自动构建、打包脚本相关内容 <!-- more -->
<The rest of contents | 余下全文>

## 构建程序
源代码通过编译生成exe的过程就是构建了，写代码的时候IDE的绿色小三角点击之后就会编译你写好的程序，没有错误的话还会顺便将生成的程序运行起来。这一过程用脚本来做的话就是自动构建了，其实IDE也是通过命令调用编译程序的，可以点击`项目->构建设置`查看。
![构建设置](https://upload-images.jianshu.io/upload_images/2756183-82e263a8fc323721.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### Qt-mingw 构建参数
```
SET PATH=%PATH%;C:\Qt\Qt5.7.0\5.7\mingw53_32\bin;
SET PATH=%PATH%;C:\Qt\Qt5.7.0\Tools\mingw530_32\bin;
SET RELEASE_DIR=%~dp0..\RunImage\release

CD %~dp0..
//Delete the last generated directory
...
if EXIST "RunImage" (
	RD /S /Q "RunImage"
)
...

mingw32-make.exe clean
qmake.exe YourProject.pro -spec win32-g++ "CONFIG+=release" && mingw32-make.exe qmake_all
mingw32-make.exe -j8
lrelease YourProject.pro

//copy you knows dll
copy /y Third\ffmpeg\bin\*.dll RunImage\release\

CD %RELEASE_DIR%
//copy Qt dll
windeployqt.exe YouProgram.exe 
```

### Qt-msvc 构建参数
```
SET PATH=%PATH%;C:\Qt\Qt5.12.0\5.12.0\msvc2017\bin;
SET PATH=%PATH%;C:\Qt\Qt5.12.0\Tools\QtCreator\bin;
SET PATH=%PATH%;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build;
SET RELEASE_DIR=%~dp0..\RunImage\release

CD %~dp0..
//Delete the last generated directory
...
if EXIST "RunImage" (
	RD /S /Q "RunImage"
)
...

//Set msvc environment
call vcvarsall.bat amd64_x86

jom.exe clean
qmake.exe YourProject.pro -spec win32-msvc "CONFIG+=qtquickcompiler" "CONFIG+=release"
jom qmake_all
jom.exe
lrelease YourProject.pro

//copy you knows dll
copy /y Third\ffmpeg\bin\*.dll RunImage\release\

CD %RELEASE_DIR%
//copy Qt dll
windeployqt.exe YouProgram.exe 
```

### 注意事项
1. 构建的时候qmake在源码路径下执行的话，生成的makefile会在源码对应pro所在文件夹内，这就相当于IDE构建中的shadow build没开的情况，如果同时需要编译多个编译器的版本的时候，上一次编译的makefile会被下一个编译器读取导致编译失败，解决方法是在源码路径外调用qmake，相当于IDE的shadow build勾选的效果
```
CD %~dp0..\..\build-test
qmake.exe %~dp0..\ComicReader.pro -spec win32-g++ "CONFIG+=release" && mingw32-make.exe qmake_all
mingw32-make.exe -j4
```
这样makefile就会生成到build-test目录中了，默认其他生成文件也会在这里，也可以在pro中指定：
```
MOC_DIR         = temp/moc
RCC_DIR         = temp/rcc
UI_DIR          = temp/ui
OBJECTS_DIR     = temp/obj
DESTDIR         = $$PWD/bin
```
2. msvc编译时要设置相关的环境，需要调用VS2017的脚本文件`call vcvarsall.bat amd64_x86` ，后面的参数是64位系统编译生成32位程序的选项

## 打包
构建并且将相关依赖的dll都拷贝到release目录后，将release目录弄个压缩包拷贝给别人就能用了，如果想弄好看点或者需要写注册表时，可以用nsis制作一个打包脚本，相关内容可以看之前的文章[Qt之打包发布程序 (NSIS)](https://www.jianshu.com/p/138606e34997)

## 自动化
前面两步按照顺序执行完后就将源码变成了最终的产品，这种固定的步骤就可以交给负责持续集成的软件来做了，只要指定好对应的环境变量还有目录位置即可，其他IDE也可以翻翻构建配置，找到命令行参数后同理。
