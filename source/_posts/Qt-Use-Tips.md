---
title: Qt使用技巧
categories:
    - Qt
tags:
    - Qt
date: 2019-09-23 22:24:24
---

<Excerpt in index | 摘要>
记录下一些使用 Qt 的小技巧<!-- more -->
<The rest of contents | 余下全文>

## 快捷操作

-   快速从函数声明创建函数定义，Alt+Enter
-   跳转到定义处，F2
-   搜索鼠标当前位置单词，F3
-   .h、.cpp 切换，F4
-   查找用过某个变量 or 函数的地方，单词上右键->Find Usages
-   定位器使用：过滤器前缀 + "空格" +   定位内容，e.g.查找名字为 save()的函数：m save
-   添加虚函数实现：类名上右键->Refactor->Insert Virtual Functions
-   查看类的继承：Ctrl+Shift+T

## 多线程编译加速（仅 Mingw）

-   对所有项目操作，在构建套件的环境中增加`MAKEFLAGS=-j8`
-   对单个项目操作，项目->构建步骤->Make 详情->Make 参数，添加：-j8

## qrc 文件不及时更新

qss 文件在 qrc 资源文件中，修改 qss 文件不重新构建不会更新。可以通过修改 qrc 文件的时间属性，让编译器能够检测到文件修改并重新编译，在 qrc 文件目录下：`touch xxx.qrc`即可。

## 构建库时安装到指定目录

1. 在 pro 配置的相关文件

```
target.path = $$LIB_INSTALL_ROOT/XXX
headers.path = $$INCLUDE_INSTALL_ROOT/XXX
XXX.h

INSTALLS += target \
headers
```

2. 构建->添加步骤->Make->参数加上 install

## Qt 信号连接重载写法

```
connect(m_pBtn, QOverload<bool>::of(&MyButton::sigClicked),this,&Widget::onClicked);
```

## 修改带 UI 的界面类继承的基类

1. .h、.cpp 将父类改成 QDialog
2. ui 文件用编辑器打开， “<widget class="QWidget"...” ，将 class 值改为 QDialog

## 语言家抽取 qml 中翻译字符串

```
//pro文件需要添加下面内容，否则不会抽取qml中字符串
lupdate_only {
SOURCES += SignIn/qml/SignInQmlView.qml
}
```

## 程序图标版本号设置

```
//pro文件中写上标记版本号+ico图标
VERSION             = 2018.7.25
win32:RC_ICONS      = main0.ico
```

## 管理员运行程序，限定在 MSVC 编译器

```
//pro文件添加
QMAKE_LFLAGS +=/MANIFESTUAC:\"level=\'requireAdministrator\' uiAccess=\'false\'\" #以管理员运行
QMAKE_LFLAGS +=/SUBSYSTEM:WINDOWS,\"5.01\" #VS2013 在XP运行
```

## 移除旧的样式

```
style()->unpolish(ui->btn);
//重新设置新的该控件的样式。
style()->polish(ui->btn);
```

## 获取类的属性

```
const QMetaObject *metaobject =object->metaObject();
int count = metaobject->propertyCount();
for (int i = 0; i < count; ++i) {
   QMetaProperty metaproperty = metaobject->property(i);
   const char *name = metaproperty.name();
   QVariant value = object->property(name);
   qDebug() << name << value;
}
```

## 根据操作系统判断加载

```
macx {
    QMAKE_INFO_PLIST = resource/app/Info.plist
    ICON = resource/app/fstl.icns
}

win32 {
    RC_FILE = resource/exe/fstl.rc
}

linux {
    target.path = /usr/bin
    INSTALLS += target
}
```
