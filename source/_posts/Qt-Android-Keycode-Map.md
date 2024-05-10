---
title: Qt键盘-Android键盘映射
categories:
  - Qt
tags:
  - Qt
  - Android
  - Keycode
date: 2019-06-17 17:31:14
---

<Excerpt in index | 摘要> 
在做远程控制的时候，需要在本地先捕获当前键盘事件，再将按的键发送给远端，在被控制端模拟对应的键盘事件来做出反应。要让程序知道按了键盘上的某个键，可以通过[键盘扫描码](https://baike.baidu.com/item/%E9%94%AE%E7%9B%98%E6%89%AB%E6%8F%8F%E7%A0%81)或者各平台的[虚拟键码](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E9%94%AE%E7%A0%81)来实现。
同一个键盘的键盘扫描码在不同平台结果都是相同的，但是不同厂家的键盘，以及接口不同的键盘(PS/2、USB) 扫描码都有可能不一样，为了减少硬件的影响还是将两个平台的虚拟键码做个映射。<!-- more -->
<The rest of contents | 余下全文>

## 映射表
只做了常见的键盘上的键，并不是全部的哈~

### 主键盘
| Qt按键 | Android按键 | Android键码 |
| --- | --- | --- |
| Qt::Key_Escape | KEYCODE_ESCAPE | 111 |
| Qt::Key_F1 | KEYCODE_F1 | 131 |
| Qt::Key_F2 | KEYCODE_F2 | 132 |
| Qt::Key_F3 | KEYCODE_F3 | 133 |
| Qt::Key_F4 | KEYCODE_F4 | 134 |
| Qt::Key_F5 | KEYCODE_F5 | 135 |
| Qt::Key_F6 | KEYCODE_F6 | 136 |
| Qt::Key_F7 | KEYCODE_F7 | 137 |
| Qt::Key_F8 | KEYCODE_F8 | 138 |
| Qt::Key_F9 | KEYCODE_F9 | 139 |
| Qt::Key_F10 | KEYCODE_F10 | 140 |
| Qt::Key_F11 | KEYCODE_F11 | 141 |
| Qt::Key_F12 | KEYCODE_F12 | 142 |
| Qt::Key_QuoteLeft | KEYCODE_GRAVE | 68 |
| Qt::Key_0 | KEYCODE_0 | 7 |
| Qt::Key_1 | KEYCODE_1 | 8 |
| Qt::Key_2 | KEYCODE_2 | 9 |
| Qt::Key_3 | KEYCODE_3 | 10 |
| Qt::Key_4 | KEYCODE_4 | 11 |
| Qt::Key_5 | KEYCODE_5 | 12 |
| Qt::Key_6 | KEYCODE_6 | 13 |
| Qt::Key_7 | KEYCODE_7 | 14 |
| Qt::Key_8 | KEYCODE_8 | 15 |
| Qt::Key_9 | KEYCODE_9 | 16 |
| Qt::Key_Minus | KEYCODE_MINUS | 69 |
| Qt::Key_Equal | KEYCODE_EQUALS |70 |
| Qt::Key_Backspace | KEYCODE_DEL | 67 |
| Qt::Key_A | KEYCODE_A | 29 |
| Qt::Key_B | KEYCODE_B | 30 |
| Qt::Key_C | KEYCODE_C | 31 |
| Qt::Key_D | KEYCODE_D | 32 |
| Qt::Key_E | KEYCODE_E | 33 |
| Qt::Key_F | KEYCODE_F | 34 |
| Qt::Key_G | KEYCODE_G | 35 |
| Qt::Key_H | KEYCODE_H | 36 |
| Qt::Key_I | KEYCODE_I | 37 |
| Qt::Key_J | KEYCODE_J | 38 |
| Qt::Key_K | KEYCODE_K | 39 |
| Qt::Key_L | KEYCODE_L | 40 |
| Qt::Key_M | KEYCODE_M | 41 |
| Qt::Key_N | KEYCODE_N | 42 |
| Qt::Key_O | KEYCODE_O | 43 |
| Qt::Key_P | KEYCODE_P | 44 |
| Qt::Key_Q | KEYCODE_Q | 45 |
| Qt::Key_R | KEYCODE_R | 46 |
| Qt::Key_S | KEYCODE_S | 47 |
| Qt::Key_T | KEYCODE_T | 48 |
| Qt::Key_U | KEYCODE_U | 49 |
| Qt::Key_V | KEYCODE_V | 50 |
| Qt::Key_W | KEYCODE_W | 51 |
| Qt::Key_X | KEYCODE_X | 52 |
| Qt::Key_Y | KEYCODE_Y | 53 |
| Qt::Key_Z | KEYCODE_Z | 54 |
| Qt::Key_Tab | KEYCODE_TAB | 61 |
| Qt::Key_CapsLock | KEYCODE_CAPS_LOCK | 115 |
| Qt::Key_Space | KEYCODE_SPACE | 62 |
| Qt::Key_BracketLeft | KEYCODE_LEFT_BRACKET | 71 |
| Qt::Key_BracketRight | KEYCODE_RIGHT_BRACKET | 72 |
| Qt::Key_Backslash | KEYCODE_BACKSLASH | 73 |
| Qt::Key_Semicolon | KEYCODE_SEMICOLON |  74 |
| Qt::Key_Apostrophe | KEYCODE_APOSTROPHE | 75 |
| Qt::Key_Return | KEYCODE_ENTER | 66 |
| Qt::Key_Comma | KEYCODE_COMMA | 55 |
| Qt::Key_Period | KEYCODE_PERIOD | 56 |
| Qt::Key_Slash | KEYCODE_SLASH | 76 |
| Qt::Key_ScrollLock | KEYCODE_SCROLL_LOCK | 116 |
| Qt::Key_Pause | KEYCODE_BREAK | 121 |
| Qt::Key_Insert | KEYCODE_INSERT | 124 |
| Qt::Key_Home | KEYCODE_MOVE_HOME | 122 |
| Qt::Key_PageUp | KEYCODE_PAGE_UP | 92 |
| Qt::Key_Delete | KEYCODE_FORWARD_DEL | 112 |
| Qt::Key_End | KEYCODE_MOVE_END | 123 |
| Qt::Key_PageDown | KEYCODE_PAGE_DOWN | 93 |
| Qt::Key_Up | KEYCODE_DPAD_UP | 19 |
| Qt::Key_Down | KEYCODE_DPAD_DOWN | 20 |
| Qt::Key_Left | KEYCODE_DPAD_LEFT | 21 |
| Qt::Key_Right | KEYCODE_DPAD_RIGHT | 22 |
| Qt::Key_Menu | KEYCODE_MENU | 82 |
| Qt::Key_Meta | --- | --- |
| Qt::Key_Control | KEYCODE_CTRL_LEFT | 113 |
| Qt::Key_Shift | KEYCODE_SHIFT_LEFT | 59 |
| Qt::Key_Alt | KEYCODE_ALT_LEFT | 57 |

### 小键盘
只标了和主键盘不同的键码，通过`event->modifiers() & Qt::KeypadModifier`检测按的键是否为小键盘上的键：

| Qt按键 | Android按键 | Android键码 |
| --- | --- | --- |
| Qt::Key_NumLock | KEYCODE_NUM_LOCK | 143 |
| Qt::Key_Slash | KEYCODE_NUMPAD_DIVIDE | 154 |
| Qt::Key_Asterisk | KEYCODE_NUMPAD_MULTIPLY | 155 |
| Qt::Key_0 | KEYCODE_NUMPAD_0 | 144 |
| Qt::Key_1 | KEYCODE_NUMPAD_1 | 145 |
| Qt::Key_2 | KEYCODE_NUMPAD_2 | 146 |
| Qt::Key_3 | KEYCODE_NUMPAD_3 | 147 |
| Qt::Key_4 | KEYCODE_NUMPAD_4 | 148 |
| Qt::Key_5 | KEYCODE_NUMPAD_5 | 149 |
| Qt::Key_6 | KEYCODE_NUMPAD_6 | 150 |
| Qt::Key_7 | KEYCODE_NUMPAD_7 | 151 |
| Qt::Key_8 | KEYCODE_NUMPAD_8 | 152 |
| Qt::Key_9 | KEYCODE_NUMPAD_9 | 153 |
| Qt::Key_Minus | KEYCODE_NUMPAD_SUBTRACT | 156 |
| Qt::Key_Plus | KEYCODE_NUMPAD_ADD | 157 |
| Qt::Key_Period | KEYCODE_NUMPAD_DOT | 158 |
| Qt::Key_Enter | KEYCODE_ENTER | 66 |

### 组合键
组合键发送一个按键列表给被控制端， 例如需要按住Shift切换的键，发送Shift + 键盘上对应的键

| Qt按键 | Android按键 | Android键码 |
| --- | --- | --- |
| Qt::Key_AsciiTilde | ---  | 59 + 68 |
| Qt::Key_Exclam | --- | 59 + 8 |
| Qt::Key_At | --- | 59 + 9 |
| Qt::Key_NumberSign | --- | 59 + 10 |
| Qt::Key_Dollar | --- | 59 + 11 |
| Qt::Key_Percent | --- | 59 +12 |
| Qt::Key_AsciiCircum | --- | 59 + 13 |
| Qt::Key_Ampersand | --- | 59 + 14 |
| Qt::Key_Asterisk | --- | 59 + 15 |
| Qt::Key_ParenLeft | --- | 59 + 16 |
| Qt::Key_ParenRight | --- | 59 + 7 |
| Qt::Key_Underscore | --- | 59 + 69 |
| Qt::Key_Plus | --- | 59 + 70 |
| Qt::Key_BraceLeft | --- | 59 + 71 |
| Qt::Key_BraceRight | --- | 59 + 72 |
| Qt::Key_Bar | --- | 59 + 73 |
| Qt::Key_Colon | --- | 59 + 74 |
| Qt::Key_QuoteDbl | --- | 59 + 75 |
| Qt::Key_Less | --- | 59 + 55 |
| Qt::Key_Greater | --- | 59 + 56 |
| Qt::Key_Question | --- | 59 + 76 |

其余的根据`event->modifiers()`来识别功能键，再加上键盘键， 例如`Ctrl+A => 113+29`。

```
    if (event->modifiers() & Qt::ControlModifier) {
        //ctrl
    }

    if (event->modifiers() & Qt::ShiftModifier) {
        //shift
    }

    if (event->modifiers() & Qt::AltModifier) {
        //alt
    }

    if (event->modifiers() & Qt::MetaModifier) {
        //win
    }
```

## 参考链接
* [Qt键盘码](https://doc.qt.io/qt-5/qt.html)
* [Android键盘码](https://www.cnblogs.com/findyou/p/5614178.html)


