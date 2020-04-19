---
title: Go语言使用Qt开发界面
categories:
  - Qt
tags:
  - Qt
  - go
date: 2020-04-19 17:16:09
---

<Excerpt in index | 摘要>
Go 语言跨平台，Qt 也跨平台，两个跨平台的框架/语言愉快地交织在了一起。我本应该获得了这种如梦一般的幸福时光才对。可是，为什么，会变成现在这样呢……（白学家梗） <!-- more -->
<The rest of contents | 余下全文>

## Go 的 UI 库

Go 语言本身是没有 UI 库的，不过有许多第三方的库支持将 Go 语言绑定到其他 UI 库，比如 Qt、GTK。本文就介绍使用这个 Qt 绑定的库，[therecipe/qt](https://github.com/therecipe/qt)。

## 环境搭建（windows）

非 windows 或者需要参数说明的可以参考官方的[wiki](https://github.com/therecipe/qt/wiki)、[windows 安装](https://github.com/therecipe/qt/wiki/Installation-on-Windows)

### Qt 安装

下载你需要的版本，[下载地址](http://download.qt.io/archive/qt/)。如果需要安装 VS 编译器什么的也可以参考下我之前的文章，[Qt5.12.0 + VS2017 环境搭建](https://longxuan.ren/2018/12/30/Qt5-12-0-VS2017-environment/)。

### therecipe/qt 安装

终端执行下面的命令:

```shell
# Go的环境变量配置，配置过一次就不用在设置了
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn

# 下载therecipe/qt库
go get -v -tags=no_env github.com/therecipe/qt/cmd/...
```

### 执行 qtsetup

下载好库后会在`%GOPATH%/bin`目录下出现几个帮助执行绑定工作的可执行文件
![Go-Use-Qt-2020-04-19-20-06-08](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Go-Use-Qt-2020-04-19-20-06-08.png)

现在还需要根据你本地的 Qt 再配置下编译环境，就是执行上面目录中的 qtsetup，但是直接执行是会有问题的：
![Go-Use-Qt-2020-04-19-20-23-26](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Go-Use-Qt-2020-04-19-20-23-26.png)

需要配置几个环境变量，`qtsetup`只需要执行一次。如果你有多个环境要切换，需要注意和编译时的环境变量保持一致，可以用 bat 临时设置下环境变量方便切换，下面就是 bat 的内容：

```bat
SET PATH=%PATH%;%GOPATH%\bin;

REM 配置Qt目录和Qt版本
SET QT_DIR=C:\Qt\Qt5.12.5
SET QT_VERSION_MAJOR=5.12.5

REM 编译32位程序，前提是你的Qt有Mingw32套件
SET GOARCH=386

REM 默认使用Mingw编译，使用MSVC编译的话开启下面的选项，qtsetup可以运行，但是后面qtdeploy会报cgo相关错误
REM SET QT_MSVC=true
REM SET PATH=%PATH%;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build;
REM call vcvarsall.bat amd64_x86

REM 语言绑定安装，只需执行一次
qtsetup.exe
```

## demo 运行

### 创建一个 Go 项目目录

终端运行:

```shell
mkdir qtwidget && cd qtwidget
go mod init
```

### main.go

在项目目录新建一个 main.go 文件，复制下面的内容:

```go
package main

import (
	"os"

	"github.com/therecipe/qt/widgets"
)

func main() {

	// needs to be called once before you can start using the QWidgets
	app := widgets.NewQApplication(len(os.Args), os.Args)

	// create a window
	// with a minimum size of 250*200
	// and sets the title to "Hello Widgets Example"
	window := widgets.NewQMainWindow(nil, 0)
	window.SetMinimumSize2(250, 200)
	window.SetWindowTitle("Hello Widgets Example")

	// create a regular widget
	// give it a QVBoxLayout
	// and make it the central widget of the window
	widget := widgets.NewQWidget(nil, 0)
	widget.SetLayout(widgets.NewQVBoxLayout())
	window.SetCentralWidget(widget)

	// create a line edit
	// with a custom placeholder text
	// and add it to the central widgets layout
	input := widgets.NewQLineEdit(nil)
	input.SetPlaceholderText("Write something ...")
	widget.Layout().AddWidget(input)

	// create a button
	// connect the clicked signal
	// and add it to the central widgets layout
	button := widgets.NewQPushButton2("and click me!", nil)
	button.ConnectClicked(func(bool) {
		widgets.QMessageBox_Information(nil, "OK", input.Text(), widgets.QMessageBox__Ok, widgets.QMessageBox__Ok)
	})
	widget.Layout().AddWidget(button)

	// make the window visible
	window.Show()

	// start the main Qt event loop
	// and block until app.Exit() is called
	// or the window is closed by the user
	app.Exec()
}

```

### 编译

编译就是执行 qtdeploy 命令，具体参数可以自己运行 qtdeploy -h 查看，这个终端文件也是在`%GOPATH%/bin`的，如果你的编译环境有多个也可以用 bat 配置下，与上面的类似，新建个 build.bat 拷贝下面的内容并运行：

```bat
SET PATH=%PATH%;%GOPATH%\bin;

REM 配置Qt目录和Qt版本
SET QT_DIR=C:\Qt\Qt5.12.5
SET QT_VERSION_MAJOR=5.12.5

REM 编译32位程序
SET GOARCH=386

REM 默认使用Mingw编译，使用MSVC编译的话开启下面的选项，qtsetup可以运行，但是后面qtdeploy会报cgo相关错误
REM SET QT_MSVC=true
REM SET PATH=%PATH%;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build;
REM call vcvarsall.bat amd64_x86

REM 语言绑定安装，只需执行一次
REM qtsetup.exe

REM 编译具体项目时运行
qtdeploy build desktop ./
```

生成的可执行文件在当前目录/deploy/windows，本体 exe 和相关的 dll 都在这个目录下

![Go-Use-Qt-2020-04-19-20-52-51](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Go-Use-Qt-2020-04-19-20-52-51.png)

注：我使用 Mingw 编译成功，但是 MSVC 编译会有 cgo 的错误，太难用了...

```shell
main.go:6:2: G:\Go\src\go_learn\qtwidget\vendor\github.com\therecipe\qt\widgets\minimal_cgo_windows_windows_386.go: invalid #cgo verb: #cgo MSCFLAGS: -nologo -Zc:wchar_t -FS -Zc:strictStrings -O2 -MD -W3 -w44456 -w44457 -w44458 -D_ALLOW_KEYWORD_MACROS -DUNICODE -D_UNICODE -DWIN32 -D_ENABLE_EXTENDED_ALIGNED_STORAGE -DQT_NO_DEBUG -DQT_WIDGETS_LIB -DQT_GUI_LIB -DQT_CORE_LIB -DNDEBUG
```

### 运行结果

demo 执行效果如下：

![Go-Use-Qt-2020-04-19-20-53-54](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Go-Use-Qt-2020-04-19-20-53-54.png)

### 项目代码

虽然没几行，不过如果懒得复制的话也可以直接下载，[Github 链接](https://github.com/Longxr/go_learn/tree/master/qtwidget)，喜欢的话记得点个 Star~

## 使用感受

- 编译的命令行配置中没有多核编译的选项，改个代码编译起码要几分钟，很难受
- 这个库没有官方文档，wiki 直接指向 Qt 的文档，所以调用都要在源码搜，看看转换函数咋写的，把 Qt 函数名大写、函数重载就名字后面加序号等，很蛋疼...
- 语法也不适应，Qt 的 connect 改成了类似方法回调的样子:`btn.ConnectClicked(func(){})`
- 支持不完善，使用 MSVC 编译会有 cgo 的错误，还是直接通过 rpc 调用或者 http 调用来让 Go 和 Qt 通信比较好
