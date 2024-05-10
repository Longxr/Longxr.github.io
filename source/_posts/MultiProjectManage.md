---
title: Qt多项目管理
categories:
  - Qt
tags:
  - Qt
  - pro
date: 2018-01-29 15:54:05
---

<Excerpt in index | 摘要> 
Qt多项目管理，pro相关设置<!-- more -->
<The rest of contents | 余下全文>



> 在做Qt的开发的时候，除了一个生成可执行程序的项目外，有时会将一些独立的模块编译成静态或动态库的形式来调用，方便后期集成到其他项目中。



## 项目类型

* 应用程序 : Application->Qt Widgets Application
* 库 : Library->C++ Library
* 子目录项目 : 其他项目->子目录项目

首先是先新建一个子目录项目，再根据需要在子目录项目中添加一个应用程序项目以及多个库的项目，子目录项目默认就只有一个.pro文件。
![工程目录](https://upload-images.jianshu.io/upload_images/2756183-85b0ec6900d21c69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## pro文件配置

### pro配置

#### 注释
从“#”开始，到这一行结束。

#### TEMPLATE
模板变量告诉qmake为这个应用程序生成哪种makefile。

| 选项      | 说明                                       |
| ------- | ---------------------------------------- |
| app     | 创建一个用于构建应用程序的Makefile（默认）。               |
| lib     | 创建一个用于构建库的Makefile。                      |
| subdirs | 创建一个用于构建目标子目录的Makefile，子目录使用SUBDIRS变量指定。 |
| aux     | 创建一个不建任何东西的Makefile。如果没有编译器需要被调用来创建目标，比如你的项目使用解释型语言写的，使用此功能。注：此模板类型只能用于Makefile-based生成器。特别是，它不会工作在vcxproj和Xcode生成器。 |
| vcapp   | 仅适用于Windows。创建一个Visual Studio应用程序项目。     |
| vclib   | 仅适用于Windows。创建一个Visual Studio库项目。        |

#### CONFIG
指定编译器选项和项目配置，值由qmake内部识别并具有特殊意义。

以下配置值控制编译标志：

| 选项                        | 说明                                       |
| ------------------------- | ---------------------------------------- |
| release                   | 项目以release模式构建。如果也指定了debug，那么最后一个生效。     |
| debug                     | 项目以debug模式构建。                            |
| debug_and_release         | 项目准备以debug和release两种模式构建。                |
| debug_and_release_target  | 此选项默认设置。如果也指定了debug_and_release，最终的debug和release构建在不同的目录。 |
| build_all                 | 如果指定了debug_and_release，默认情况下，该项目会构建为debug和release模式。 |
| autogen_precompile_source | 自动生成一个.cpp文件，包含在.pro中指定的预编译头文件。          |
| ordered                   | 使用subdirs模板时，此选项指定应该按照目录列表的顺序处理它们。       |
| precompile_header         | 可以在项目中使用预编译头文件的支持。                       |
| warn_on                   | 编译器应该输出尽可能多的警告。如果也指定了warn_off，最后一个生效。    |
| warn_off                  | 编译器应该输出尽可能少的警告。                          |
| exceptions                | 启用异常支持。默认设置。                             |
| exceptions_off            | 禁用异常支持。                                  |
| rtti                      | 启用RTTI支持。默认情况下，使用编译器默认。                  |
| rtti_off                  | 禁用RTTI支持。默认情况下，使用编译器默认。                  |
| stl                       | 启用STL支持。默认情况下，使用编译器默认。                   |
| stl_off                   | 禁用STL支持。默认情况下，使用编译器默认。                   |
| thread                    | 启用线程支持。当CONFIG包括qt时启用，这是缺省设置。            |
| c++11                     | 启用c++11支持。如果编译器不支持c++11这个选项，没有影响。默认情况下，支持是禁用的。 |
| c++14                     | 启用c++14支持。如果编译器不支持c++14这个选项，没有影响。默认情况下，支持是禁用的。 |

#### DEFINES
qmake添加这个变量的值作为编译器C预处理器宏(-D选项)。

#### INCLUDEPATH
指定编译项目时应该被搜索的#include目录。

#### DEPENDPATH
指定程序编译时依赖的相关路径。

#### DESTDIR
指定在何处放置目标文件。

#### TARGET
指定目标文件的名称。默认情况下包含的项目文件的基本名称。

#### OUT_PWD
指定构建目录。

#### MOC_DIR 
指定来自moc的所有中间文件放置的目录（含Q_OBJECT宏的头文件转换成标准.h文件的存放目录）。

#### OBJECTS_DIR
指定所有中间文件.o（.obj）放置的目录。

#### RCC_DIR
指定Qt资源编译器输出文件的目录（.qrc文件转换成qrc_*.h文件的存放目录）。

#### LIBS
指定链接到项目中的库列表。-L后是库文件的目录，-l后是具体的库的名字(后缀不用加)。例如：
`LIBS += -L$$PWD/../../../ThirdLib/winapi  -lWSock32 `

#### RC_ICONS
仅适用于Windows，指定的图标应该包含在一个生成的.rc文件里。如果RC_FILE 和RES_FILE变量都没有设置这才可利用。

#### TRANSLATIONS
指定包含用户界面翻译文本的翻译(.ts)文件列表。

#### 平台相关性处理
为Windows平台加的依赖平台的文件的简单的作用域看起来就像这样：
`win32 { SOURCES += hello_win.cpp }`

### 子项目pro文件
每次添加一个子项目就会在SUBDIRS项目加上一个项目名，子目录项目的*TEMPLATE = subdirs*。*CONFIG += ordered*是让项目按照顺序编译，由于主程序依赖于对应的库，一般都是把MainApp放在SUBDIRS的最后。

### pri文件
对于一些库和主程序都需要的目录设置 (构建目录、obj生成目录、moc文件生成目录等)，可以统一写在一个pri文件中：
```
CONFIG(release, debug|release) {
    BuildType=release
    CONFIG += warn_off
} else {
    BuildType=debug
    DEFINES += __DEBUG
}

INTERMEDIATE_DIR = $$PWD/../Intermediate/$$BuildType

DESTDIR         = $$PWD/../RunImage/$$BuildType
OUT_PWD         = $$INTERMEDIATE_DIR/$$TARGET
MOC_DIR         = $$INTERMEDIATE_DIR/$$TARGET
OBJECTS_DIR     = $$INTERMEDIATE_DIR/$$TARGET
RCC_DIR         = $$INTERMEDIATE_DIR/$$TARGET
UI_DIR          = $$INTERMEDIATE_DIR/$$TARGET
```

在需要引用的pro文件中：
`include($$PWD/../../../Path.pri)`

### 主程序pro文件
主程序的*TEMPLATE = app*。除了常规的pro设置外，要记得将自己编的库的头文件目录加上，并将生成的dll通过LIBS链接：
```
#library
INCLUDEPATH += \
    $$PWD/../../Libraries \

LIBS += \
    -L$$DESTDIR -lComponent -lcommonLibs \
```

### 库pro文件
库的*TEMPLATE = lib*。库在通过Creator新建的时候会生成一个xxx_global.h，定义了导出库的相关信息：
```
#include <QtCore/qglobal.h>

#if defined(TESTLIB_LIBRARY)
#  define TESTLIBSHARED_EXPORT Q_DECL_EXPORT
#else
#  define TESTLIBSHARED_EXPORT Q_DECL_IMPORT
#endif
```
在库中要作为接口的类记得加上对应的宏：
`class TESTLIBSHARED_EXPORT TestLib : public QObject`
静态库貌似不需要这个，待测试...

## 参考资料
- [Qt之pro配置详解](http://blog.csdn.net/liang19890820/article/details/51774724)
- [QT中PRO文件写法的详细介绍](http://blog.csdn.net/adriano119/article/details/5878169)


