---
title: Qt Widget与原生窗口的对比
categories:
  - Qt
tags:
  - Qt
  - QWidget
  - parent
date: 2020-06-08 11:05:25
---

<Excerpt in index | 摘要>
QWidget 和原生窗口的一些对比<!-- more -->
<The rest of contents | 余下全文>

# 原生窗口的从属关系

包括父/子（parent-child）关系、拥有/被拥有（owner-owned）关系及兄弟（siblings）关系。

## 父子关系

1. 子窗口的位置坐标都是相对于父窗口客户区的左上角（upper-left corner）计算的。
2. 子窗口会把它的 notify 消息发送到父窗口。
3. 子窗口只能显示在它的父窗口的客户区中，超出父窗口客户区的部分将被裁减掉；父窗口被隐藏时，它的所有子窗口也被隐藏；
4. 最小化父窗口不影响子窗口的可见状态，子窗口会随着父窗口被最小化，但是它的 WS_VISIBLE 属性不变。
5. 父窗口被销毁的时候，它的所有子窗口都会被销毁。

## 拥有/被拥有关系

顶层窗口之间可以存在 owner-owned 关系。owner-owned 关系对窗口可见性的影响为：owned 窗口永远显示在 owner 窗口的前面；当 owner 窗口最小化的时候，它所拥有的窗口都会被隐藏；隐藏 owner 窗口不影响它所拥有的窗口的可见状态。根据最后这一点，如果窗口 A 拥有窗口 B,窗口 B 拥有窗口 C,则当窗口 A 最小化的时候，窗口 B 被隐藏，但是窗口 C 还是可见的。当 owner 窗口被销毁的时候，它所拥有的窗口都会被销毁。

## 兄弟关系

同一个父窗口的所有直属子窗口之间是兄弟关系，也就是相互平等，没有主从之分。窗口管理器用链表（linked list）来管理每个父窗口的直属子窗口，这个链表叫子窗口链（child window list）。
调用 GetWindow 函数时使用 GW_HWNDPREV 或 GW_HWNDNEXT 标志可访问子窗口链中的前一个或后一个窗口；使用 GW_HWNDFIRST 或 GW_HWNDLAST 标志可访问子窗口链中的第一个或最后一个窗口。

# QWidget 的丛属关系

## 父子关系

在创建 Qt 对象的时候，一般都会指定一个 parent。指定了 parent 后，当父对象删除的时候会将其子对象也一并删除，所以在 Qt 中经常是看到 new 却没有对应的 delete。不过指定了 parent 之后的 widget 默认就不是原生的 native widget 了，通过 spy++是看不到对应的窗口的，创建的是 Qt 的 Alien Widget，只能看到顶层的 MainWindow:

![Qt-Widget-NativeWindow-2020-06-08-12-12-08](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Widget-NativeWindow-2020-06-08-12-12-08.png)

但是在父子关系下，特性和原生窗口差不多：

1. 子 Widget 的位置坐标是相对于父窗口的左上角计算的。
2. 子 Widget 的事件默认不会再往父 Widget 传递
3. 子 Widget 只能显示在它的父窗口的客户区中，超出父窗口客户区的部分将被裁减掉；父窗口被隐藏时，它的所有子 Widget 也被隐藏。
4. 父 Widget 被销毁时，子 Widget 也会被销毁。

在某些使用第三方库的情况下，可能需要一个子 Widget 的句柄，可以调用子 Widget 的`winId()`方法，不过调用之后同一个 parent 下的 widget 都会变成原生窗口。

![Qt-Widget-NativeWindow-2020-06-08-13-46-37](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Widget-NativeWindow-2020-06-08-13-46-37.png)

## 兄弟关系

同一个 parent 下的子 widget，创建后会放在父 Widget 的创建栈 （widget's stack） 中。后创建的显示在最前面，如果需要把之前创建的 widget 显示在前面可以调用`raise()`方法。

如果创建 widget 时没有指定 parent，那么创建的就是 native 的顶层窗口，直接就有句柄。此时调用`raise()`方法也有效。

![Qt-Widget-NativeWindow-2020-06-08-13-58-02](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-Widget-NativeWindow-2020-06-08-13-58-02.png)

# 参考链接

- [QWidget 之 Alien 与 Native 小记](https://blog.csdn.net/dbzhang800/article/details/7006270)
- [从 QWindow 到 QWidget(Qt5)](https://blog.csdn.net/dbzhang800/article/details/7010114)
- [窗口之间的主从关系与 Z-Order](https://www.cnblogs.com/dhatbj/p/3288152.html)
