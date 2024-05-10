---
title: Mac操作小技巧
categories:
  - 操作技巧
tags:
  - Mac
  - 快捷键
date: 2017-04-13 11:00:32
---

<Excerpt in index | 摘要> 
常用快捷键
文件用Sublime打开
Finder与终端交互<!-- more -->
<The rest of contents | 余下全文>

## 常用快捷键
> 有些事情自带快捷键就能做到，不用去配置服务、终端敲代码、另外下软件什么的

### 快捷键图标
⌘——Command ()
⌃ ——Control
⌥——Option (Alt)
⇧——Shift
⇪——Caps Lock
#### 终端清屏：command＋K
#### 文件预览：空格
#### 中文输入法中英文切换：CapsLock
#### 拷贝文件路径：Command + Option + C
#### 显示/隐藏文件：Command + Shift + .
#### 全屏截图：Command+Shift+3
#### 指定区域截图：Command+Shift+4
#### 指定窗口截图：Command+Shift+4，空格+鼠标单击指定窗口
#### 剪切移动文件：Command+C 拷贝文件，Command+Option+V 剪切文件

## 文件用Sublime打开
> 经常面对各种格式的文本文件，分别设置打开方式太麻烦，统一用Sublime打开，添加到右键菜单功能

1. 打开 `Automator` 这个程序（Finder应用程序中找或者Spotlight直接搜索），在弹出的菜单中选择 __服务__：

  ![打开.png](https://upload-images.jianshu.io/upload_images/2756183-7cae0171e6be0b99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 在左上角的搜索框搜索 Finder 把打开Finder项目拖动到右边：

  ![拖动.png](https://upload-images.jianshu.io/upload_images/2756183-fb3b74222ac8ac62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 右上角服务设置如图：

  ![服务设置.png](https://upload-images.jianshu.io/upload_images/2756183-f46b8a18118e66b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.效果图：

![右键菜单.png](https://upload-images.jianshu.io/upload_images/2756183-18834771e83a532b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  如果设置的服务出现在二级菜单，可以在`系统偏好设置`->`键盘`->`快捷键`->`服务`中去掉不需要的选项

## Finder与终端交互
### 终端到Finder
  终端目录下输入：`open .`

### Finder到终端
> 默认的系统偏好设置中的服务中有一个在文件夹当前位置打开终端的选项，但是因为换了[iTerm2](https://www.iterm2.com)，所以并不适用

  使用软件：[Go2shell](http://zipzapmac.com/Go2Shell)
  下载后安装，选择你使用的终端，点下  `install Go2Shell from Finder` 即可:
  
  ![安装.png](https://upload-images.jianshu.io/upload_images/2756183-7e85a93ff9470903.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![gotoshell.png](https://upload-images.jianshu.io/upload_images/2756183-19460211cf401452.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 参考文章
[把Sublime添加到Mac右键菜单](http://www.jianshu.com/p/e2f897933c56)

[Finder与iTerm2互相打开的方法](http://www.rxna.cn/post/wiki/finderyu-iterm2hu-xiang-da-kai-de-fang-fa)


