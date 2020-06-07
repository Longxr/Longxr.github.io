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

ATOM   GlobalAddAtom(//用来获取热键的唯一标识
LPCTSTR   lpString   //自己设定的一个字符串
);

```

注册时按键的虚拟码可以去[官网查询](https://docs.microsoft.com/zh-cn/windows/win32/inputdev/virtual-key-codes)。
注册按键的函数一般都是字符串的形式，可以自己写个转换函数，也可以使用 Qt 的`QKeySequence`先得到热键的 Qt 的键盘码，再映射到 windwos 的虚拟码：

```C++
quint32 nativeKeycode(Qt::Key key) {//键盘码映射
    switch (key)
    {
    case Qt::Key_Escape:
        return VK_ESCAPE;
    case Qt::Key_Tab:
    case Qt::Key_Backtab:
        return VK_TAB;
    case Qt::Key_Backspace:
        return VK_BACK;
    case Qt::Key_Return:
    case Qt::Key_Enter:
        return VK_RETURN;
    case Qt::Key_Insert:
        return VK_INSERT;
    case Qt::Key_Delete:
        return VK_DELETE;
    case Qt::Key_Pause:
        return VK_PAUSE;
    case Qt::Key_Print:
        return VK_PRINT;
    case Qt::Key_Clear:
        return VK_CLEAR;
    case Qt::Key_Home:
        return VK_HOME;
    case Qt::Key_End:
        return VK_END;
    case Qt::Key_Left:
        return VK_LEFT;
    case Qt::Key_Up:
        return VK_UP;
    case Qt::Key_Right:
        return VK_RIGHT;
    case Qt::Key_Down:
        return VK_DOWN;
    case Qt::Key_PageUp:
        return VK_PRIOR;
    case Qt::Key_PageDown:
        return VK_NEXT;
    case Qt::Key_F1:
        return VK_F1;
    case Qt::Key_F2:
        return VK_F2;
    case Qt::Key_F3:
        return VK_F3;
    case Qt::Key_F4:
        return VK_F4;
    case Qt::Key_F5:
        return VK_F5;
    case Qt::Key_F6:
        return VK_F6;
    case Qt::Key_F7:
        return VK_F7;
    case Qt::Key_F8:
        return VK_F8;
    case Qt::Key_F9:
        return VK_F9;
    case Qt::Key_F10:
        return VK_F10;
    case Qt::Key_F11:
        return VK_F11;
    case Qt::Key_F12:
        return VK_F12;
    case Qt::Key_F13:
        return VK_F13;
    case Qt::Key_F14:
        return VK_F14;
    case Qt::Key_F15:
        return VK_F15;
    case Qt::Key_F16:
        return VK_F16;
    case Qt::Key_F17:
        return VK_F17;
    case Qt::Key_F18:
        return VK_F18;
    case Qt::Key_F19:
        return VK_F19;
    case Qt::Key_F20:
        return VK_F20;
    case Qt::Key_F21:
        return VK_F21;
    case Qt::Key_F22:
        return VK_F22;
    case Qt::Key_F23:
        return VK_F23;
    case Qt::Key_F24:
        return VK_F24;
    case Qt::Key_Space:
        return VK_SPACE;
    case Qt::Key_Asterisk:
        return VK_MULTIPLY;
    case Qt::Key_Plus:
        return VK_ADD;
    case Qt::Key_Comma:
        return VK_SEPARATOR;
    case Qt::Key_Minus:
        return VK_SUBTRACT;
    case Qt::Key_Slash:
        return VK_DIVIDE;

        // numbers
    case Qt::Key_0:
    case Qt::Key_1:
    case Qt::Key_2:
    case Qt::Key_3:
    case Qt::Key_4:
    case Qt::Key_5:
    case Qt::Key_6:
    case Qt::Key_7:
    case Qt::Key_8:
    case Qt::Key_9:
        return key;

        // letters
    case Qt::Key_A:
    case Qt::Key_B:
    case Qt::Key_C:
    case Qt::Key_D:
    case Qt::Key_E:
    case Qt::Key_F:
    case Qt::Key_G:
    case Qt::Key_H:
    case Qt::Key_I:
    case Qt::Key_J:
    case Qt::Key_K:
    case Qt::Key_L:
    case Qt::Key_M:
    case Qt::Key_N:
    case Qt::Key_O:
    case Qt::Key_P:
    case Qt::Key_Q:
    case Qt::Key_R:
    case Qt::Key_S:
    case Qt::Key_T:
    case Qt::Key_U:
    case Qt::Key_V:
    case Qt::Key_W:
    case Qt::Key_X:
    case Qt::Key_Y:
    case Qt::Key_Z:
        return key;

    default:
        return 0;
    }
}
```
