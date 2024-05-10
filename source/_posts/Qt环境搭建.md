---
title: Qt环境搭建
categories:
  - 环境搭建
tags:
  - Qt
date: 2017-07-29 16:21:02
---

<Excerpt in index | 摘要> 
在工作之前也是没有想过会用到Qt，既然任务交待下来了也没辙了╰（￣ ￣）╭<!-- more -->
<The rest of contents | 余下全文>

## 注册

不管用哪种安装方法，装的时候都要登录下Qt的账号，所以还是先去[官网注册](https://login.qt.io/register)一个账号吧

## 下载
下载这里就比较蛋疼了，官网找开源版本之前点来点去找了一圈，因为有针对企业的版本拿来卖的，开源版还要填什么协议啥的

### 最新版本下载
这里是[下载地址](https://www.qt.io/download-open-source/)，可以点最下面的`View All Downloads`展开不同平台的版本：

![最新版下载](http://upload-images.jianshu.io/upload_images/2756183-d7203a058bd63135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再选择自己要的版本就好了。这里要吐槽下，Online是在线安装，真心慢....

### 历史版本下载
> 在和其他人协作的时候，就需要开发环境什么的保持一致了，要不然指不定报个什么鬼错出来...

这里是[下载地址](http://download.qt.io/official_releases/qt/)，目前常用的版本都在这里吧，选自己需要的就行：

![历史版本下载](http://upload-images.jianshu.io/upload_images/2756183-ebdbede0e93bcdb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### windows版本选择
windows根据编译器的不同有两个选项，一个是VS，另一个是MinGw：

![windows版本选择](http://upload-images.jianshu.io/upload_images/2756183-fe5c5f3a650b74e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### VS版
除了安装Qt，还要装好VS，另外还要[下载vs-addin](http://download.qt.io/official_releases/vsaddin/)：

![相关下载](http://upload-images.jianshu.io/upload_images/2756183-60cb36247b82aaef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作为一个懒人，我选择了底下这种更方便的方式....

### MinGw版
如果是第一次安装的话，推荐MinGw版，自带编译器，装上就能用，不需要额外配置：

![MinGw版](http://upload-images.jianshu.io/upload_images/2756183-06c8d0c5d6625f61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 安装
下载好后，点击安装会提示登录你的Qt账号，登录后就可以安装了：

![安装](http://upload-images.jianshu.io/upload_images/2756183-d1f66b8758c5ea93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Qt会自动将文档也装好的，可以在开始菜单下看到文档助手Assistant：

![文件目录](http://upload-images.jianshu.io/upload_images/2756183-668f30c9762bad81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以在写代码的时候在相关类名上按下F1，也会跳到相关文档（这个有时候不好使，在助手里搜比较方便）
