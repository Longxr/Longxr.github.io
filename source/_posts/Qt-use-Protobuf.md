---
title: Qt-msvc使用Protocol Buffers
categories:
  - Qt
tags:
  - Qt
  - Protobuf
  - msvc
date: 2019-07-04 15:19:43
---

<Excerpt in index | 摘要> 
Qt + VS2017 使用Protocol Buffers<!-- more -->
<The rest of contents | 余下全文>

## 简介
Protocol Buffer是google 的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

## 准备工作
1. Qt-msvc相关配置，我的是Qt5.12.2 + VS2017，[相关环境搭建教程戳这里](https://www.jianshu.com/p/ac22d511aea6)。
2. Protocol Buffer源码下载，我使用的是[protobuf-cpp-3.8.0.zip](https://github.com/protocolbuffers/protobuf/releases/download/v3.8.0/protobuf-cpp-3.8.0.zip)。
3. CMake安装，我用的是3.13.4 64bit, [相关下载链接](https://cmake.org/download/)。

## 编译Protocol Buffer
### 配置CMake
1. 打开CMake (cmake-gui)，设置好源码路径和build路径。
2. 点击左下角的configure，弹出的对话框中编译32位就选择，`Visual Studio 15 2017`;64位就选择Visual Studio 15 2017 Win64。
3. configure选好后，就会有下图红色部分的配置信息，其中蓝色的项是确定编译生成动态库还是静态库的，要生成动态库就勾选上，其他配置默认不变。
4. 点击左下角的Generate，生成VS的工程文件。
5. 点击左下角的Open Project，打开对应的VS工程。
![CMake配置](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_01.png)
![Configure选择](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_02.png)

### VS中生成库文件
1. release还是debug在菜单栏切换
![releasse、debug切换](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_03.png)
2. 大工程中还包含了些测试工程，必须生成的有两个，也可以偷懒直接右键解决方案->生成解决方案。
![不同工程](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_04.png)
3. 生成的文件就在之前设置的CMakebuild目录下的Debug、Release文件夹下。

## 使用Protocol Buffer
### 库文件整理
1. 新建一个protobuf文件夹，再建include、bin、lib三个子文件夹。
2. 将源码文件夹下的`protobuf-3.8.0\src\google`这个google文件夹拷贝至include中。
3. libprotobufd.dll、libprotocd.dll、protoc.exe放到bin中。
4. libprotobufd.lib、libprotocd.lib放到lib中。

### 定义proto文件
新建一个person.proto文件，文件内容如下：
```
package tutorial;

message Person {
  required int32 id = 1;
  required string name = 2;
  optional string email = 3;
}
```
### 生成C++代码
将person.proto放至bin目录下，在命令行输入下方内容或者新建个bat输入下方内容保存并运行：
```
protoc --cpp_out=./ person.proto
```
![TIM截图20190704145611.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_05.png)

### Qt中使用Protocol Buffer
新建一个控制台工程，将上面生成的person.pb.h、person.pb.cc加入工程，并修改main.cpp和.pro文件

#### pro文件
```
QT -= gui

CONFIG += c++11 console
CONFIG -= app_bundle

# The following define makes your compiler emit warnings if you use
# any Qt feature that has been marked deprecated (the exact warnings
# depend on your compiler). Please consult the documentation of the
# deprecated API in order to know how to port your code away from it.
DEFINES += QT_DEPRECATED_WARNINGS

# You can also make your code fail to compile if it uses deprecated APIs.
# In order to do so, uncomment the following line.
# You can also select to disable deprecated APIs only up to a certain version of Qt.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

DEFINES += PROTOBUF_USE_DLLS

INCLUDEPATH += $$PWD/protobuf/include
LIBS += -L$$PWD/protobuf/bin/debug -L$$PWD/protobuf/lib/debug -llibprotobufd

SOURCES += \
        main.cpp \
    person.pb.cc

# Default rules for deployment.
qnx: target.path = /tmp/$${TARGET}/bin
else: unix:!android: target.path = /opt/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target

HEADERS += \
    person.pb.h

```

#### main.cpp
```
#include <QCoreApplication>
#include <QDataStream>
#include <QFile>
#include <string>
#include <sstream>
#include <iostream>
#include <QDebug>
#include "person.pb.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    GOOGLE_PROTOBUF_VERIFY_VERSION;

    //set person protobuf
    tutorial::Person person;
    person.set_id(123456);
    person.set_name("Mark");
    person.set_email("mark@example.com");

    //write
    QFile file_out(QCoreApplication::applicationDirPath() + "/person.pb");
    if(!file_out.open(QIODevice::WriteOnly)) {
        qDebug() << "can not open file";
    }

    std::ostringstream streamOut;
    person.SerializeToOstream(&streamOut);
    QByteArray byteArray(streamOut.str().c_str());

    QDataStream out(&file_out);
    out << byteArray.length();
    out.writeRawData(byteArray.data(), byteArray.length());
    file_out.close();

    //read
    QFile file_in(QCoreApplication::applicationDirPath() + "/person.pb");
    if(!file_in.open(QIODevice::ReadOnly)) {
         qDebug() << "can not open file";
    }

    QDataStream in(&file_in);
    int dataLength = 0;
    in >> dataLength;

    QByteArray byteArray2;
    byteArray2.resize(dataLength);
    in.readRawData(byteArray2.data(), dataLength);

    tutorial::Person person2;
    if (!person2.ParseFromArray(byteArray2.data(), byteArray2.size())) {
      qDebug() << "Failed to parse person.pb.";
    }

    qDebug() << "ID: " << person.id();
    qDebug() << "name: " << QString::fromStdString(person.name());
    if (person.has_email()) {
        qDebug() << "e-mail: " << QString::fromStdString(person.email());
    }
    file_in.close();

    return a.exec();
}
```
![运行结果](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-use-Protobuf_06.png)

Protocol Buffers数据写入和读取的简单应用就完成了。

## 参考链接
* [Google Protocol Buffer在windows下的配置](https://blog.csdn.net/yizhou2010/article/details/80610881)
* [protobuf的编译和使用，在windows平台上](https://blog.csdn.net/hp_cpp/article/details/81561310)

