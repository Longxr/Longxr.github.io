---
title: Qt全局热键实现
categories:
  - Qt
tags:
  - Shortcut
date: 2020-06-06 12:11:24
---

<Excerpt in index | 摘要>
Qt 中没有实现好的跨平台全局热键，需要自己调用系统 API 来实现。<!-- more -->
<The rest of contents | 余下全文>

# 监听系统原生事件

Qt 作为跨平台的框架，实现的功能是在每个平台都共有的部分，而有的功能可能由于某些原因不能跨平台就没有，这时候就需要调用系统原生的 API，比如全局热键的功能、电池休眠事件监听等。

## 窗口的原生事件

窗口的原生事件可以通过重载`QWidget::nativeEvent(const QByteArray &eventType, void *message, long *result)`来实现。在函数中返回 true 会停止传递，如果返回 false 会继续由 Qt 处理事件。当窗口有句柄的时候，触发了平台原生的事件就会调用此函数。由 eventType 标识本机平台事件，这些事件的参数在 message 中传递。不同平台的 eventType 、message 如下：

| Platform | Event Type Identifier | Message Type           | Result Type |
| -------- | --------------------- | ---------------------- | ----------- |
| Windows  | "windows_generic_MSG" | MSG \*                 | LRESULT     |
| macOS    | "NSEvent"             | NSEvent \*             |             |
| XCB      | "xcb_generic_event_t" | xcb_generic_event_t \* |             |

下面举个例子，比如当前窗口在做一些操作，笔记本合上盖子进入休眠，要监听电池相关的事件：

```C++
bool MainWidget::nativeEvent(const QByteArray &eventType, void *message, long *result)
{
    if (eventType == "windows_generic_MSG" || eventType == "windows_dispatcher_MSG") {
        MSG* msg = static_cast<MSG*>(message);
        if (msg->message == WM_POWERBROADCAST) {
            if(msg->wParam == PBT_APMSUSPEND) {
                qDebug("系统休眠");
            }
            else if(msg->wParam == PBT_APMRESUMESUSPEND || msg->wParam == PBT_APMRESUMEAUTOMATIC) {
                qDebug("系统唤醒");
            }
        }
    }

    return QWidget::nativeEvent(eventType, message, result);
}
```

## 应用级原生事件

像全局热键这样的事件是应用级别的，事件的监听就要放在 app 上。通过继承`QAbstractNativeEventFilter`来实现自己的原生事件过滤器并安装到 app 上即可。当有原生事件触发后就会调用事件过滤器的`virtual bool nativeEventFilter(const QByteArray &eventType, void *message, long *)`函数。
要处理热键的话重写函数如下：

```C++
bool MyWinEventFilter::nativeEventFilter(const QByteArray &eventType, void *message, long *)
{
   if(eventType == "windows_generic_MSG") {
        MSG *msg = static_cast<MSG *>(message);
        if(msg->message == WM_HOTKEY) {
            //判断是否为指定热键按下
        }

    }
    return false;
}
```

# 热键的注册

上面处理好了事件发生后的步骤，还需要告诉系统什么哪些算是热键。注册用到的原生 API 有：

```C++
BOOL   RegisterHotKey(
HWND   hWnd,         //响应该热键的窗口句柄, 由于是应用级的，可以不传
Int   id,            //该热键的唯一标识
UINT   fsModifiers,  //该热键的辅助按键，Ctrl、Alt、Shift
UINT   vk            //该热键的键值
);

BOOL UnregisterHotKey(
HWND hWnd,           //响应该热键的窗口句柄,同不传
Int id               //该热键的唯一标识
);

```

注册时按键的虚拟码可以去[官网查询](https://docs.microsoft.com/zh-cn/windows/win32/inputdev/virtual-key-codes)。
注册按键的函数一般都是字符串的形式，可以自己写个转换函数，也可以使用 Qt 的`QKeySequence`先得到热键的 Qt 的键盘码，再映射到 windwos 的虚拟码。

# 具体实现

具体代码就不贴了，在 Github 上有个封装好各个平台的全局热键的仓库，[QHotkey](https://github.com/Skycoder42/QHotkey)

windows 相关实现在`QHotkey/QHotkey/qhotkey_win.cpp`中，里面就是调用 `RegisterHotKey` 来实现的。
