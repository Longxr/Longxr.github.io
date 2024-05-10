---
title: Sublime通用插件记录
categories:
  - 环境搭建
tags:
  - Sublime
date: 2018-08-30 09:41:04
---

<Excerpt in index | 摘要> 
偶尔重置系统，前端语言还经常换，就记几个常用的插件得了 (装多了快捷键冲突也懒得改)，剩下的根据需求再添加 <!-- more -->
<The rest of contents | 余下全文>

### 本体
Sublime免费的，只是偶尔弹窗一下，相当良心了
[官网地址](https://www.sublimetext.com/3)，根据你的系统选择下载

### Package Control
安装插件之前要先装下[Package Control](https://packagecontrol.io/installation)

#### 自动安装
使用Ctrl+`快捷键或者通过View->Show Console菜单打开终端，粘贴如下代码：
```
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
如果终端安装报错的话，就改用手动安装

#### 手动安装
1. 点击`Preferences > Browse Packages`菜单
2. 进入打开的目录的上层目录，然后再进入`Installed Packages/`目录
3. 下载[Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package)并复制到`Installed Packages/`目录
4. 重启Sublime Text

#### 使用
快捷键：**Ctrl+Shift+P**，调出菜单，输入`pci`，再输入你要安装的插件名字即可
![Package Control](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Sublime-Common-Plugin_01.png)


### 插件安装

| 插件名 | 说明 |
| --------- | ------ |
| Package Control | 管理插件的插件 |
| ChineseLocalization | 中文菜单 |
| Emmet | 前端神器 |
| DocBlocker | 文档注释插件 |
| SideBarEnhancements | 侧边栏增强插件 |
| BrackeHighlighter | 匹配括号标签引号的插件 |
| AdvancedNewFile | 新建文件插件，快捷键Ctrl+Alt+N |
| Boxy Theme，A File Icon | 主题图标等配置 |

### 个人配置
Sublime的配置有default和user，前者是默认配置，一般个人修改都是将修改内容粘贴到对应的user配置中即可
![配置修改](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Sublime-Common-Plugin_02.png)


#### 全局设置

```
{

    "ensure_newline_at_eof_on_save": true,

    "font_size": 14,

    "highlight_line": true,

    "update_check":false

}
```

#### 添加Sublime到右键菜单

打开regedit.exe注册表

添加路径：`HKEY_CLASSES_ROOT\*\shell\Sublime\Command`，添加命令：`"D:\Sublime Text Build 3103\sublime_text.exe" "%1"`

#### AdvancedNewFile配置
```
{

 "default_root": "current"

}
```

#### SideBarEnhancements 配置

##### 按键绑定(按F1、F2直接在浏览器中打开当前文件)
```
//***********************************Win版*****************************

[

       /*{ "keys": ["alt+f12"],

            "command": "side_bar_open_in_browser",

            "args":{"paths":[], "type":"production", "browser":""}

        },*/

          { "keys": ["ctrl+shift+c"], "command": "copy_path" },

          //chrome

    { "keys": ["f1"], "command": "side_bar_files_open_with",

            "args": {

                "paths": [],

                "application": "C://Program Files (x86)//Google//Chrome//Application//chrome.exe",

                "extensions":".*"

            }

     },

    //firefox

    { "keys": ["f2"], "command": "side_bar_files_open_with",

             "args": {

                "paths": [],

                "application": "D://Firefox//firefox.exe",

                "extensions":".*" //匹配任何文件类型

            }

    },

    //ie

     { "keys": ["f3"], "command": "side_bar_files_open_with",

             "args": {

                "paths": [],

                "application": "C://Program Files//Internet Explorer//iexplore.exe",

                "extensions":".*"

            }

    },

    ]

//***********************************Mac版*****************************

[

       /*{ "keys": ["alt+f12"],

            "command": "side_bar_open_in_browser",

            "args":{"paths":[], "type":"production", "browser":""}

        },*/

          { "keys": ["ctrl+shift+c"], "command": "copy_path" },

    //safari

    { "keys": ["f1"], "command": "side_bar_files_open_with",

            "args": {

                "paths": [],

                "application": "safari.app",

                "extensions":".*"

            }

     },

    //chrome

    { "keys": ["f2"], "command": "side_bar_files_open_with",

             "args": {

                "paths": [],

                "application": "Google Chrome.app",

                "extensions":".*" //匹配任何文件类型

            }

    },

    ]

//***********************************ubuntu版*****************************

[

  /*{ "keys": ["alt+f12"],

  "command": "side_bar_open_in_browser",

  "args":{"paths":[], "type":"production", "browser":""}

  },*/

  { "keys": ["ctrl+shift+c"], "command": "copy_path" },

  //chrome

  { "keys": ["f1"], "command": "side_bar_files_open_with",

  "args": {

  "paths": [],

  "application": "/usr/bin/google-chrome",

  "extensions":".*" //匹配任何文件类型

  }

  },

  ]
```

### 插件使用

#### Emmet插件使用

输入相应指令格式，按Tab或Ctrl+E实现代码补全

#### DocBlocker插件使用

- /* : 回车创建一个代码块注释

- /** : 回车创建一个代码块注释，如果在函数or变量前一行会自动添加函数、变量参数说明等，继续回车会自动加上*，再输入@后可以再根据需要添加(自动补全)需要的注释内容

- 变量前输入/**按Enter，补全变量多行注释

![注释插件.gif](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Sublime-Common-Plugin_03.gif)


如果输入了一个行注释，再按Ctrl+Enter，注释将变成酱紫

```
// Foo bar baz<<Ctrl+Enter>>

-- becomes

/////////////////
// Foo bar baz //
/////////////////
```

