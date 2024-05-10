---
title: Hexo个人博客搭建
categories:
  - 环境搭建
tags:
  - blog
  - Hexo
date: 2017-04-07 16:41:30
---

<Excerpt in index | 摘要> 
虽然之前搭建好了自己的[博客](http://longxuan.ren)，然后闲置了一个月...没有写文章的习惯啊！！以后每周至少写点东西先养成个习惯Orz。<!-- more -->
<The rest of contents | 余下全文>


## 步骤

1. 准备放博客的空间，我是选择的github的github pages，免费，有逼格

2. 配置Hexo环境，后期可以通过配置Travis CI自动化部署

3. 绑定自己的域名，虽然github.io也挺好的，没事就喜欢折腾2333

## github pages设置

首先得有一个[github账号](https://github.com)，账号申请就略过了。

在账号下新建一个仓库，仓库命名为yourname.github.io。

![github新建仓库.png](https://upload-images.jianshu.io/upload_images/2756183-2f7eb03ffa9cda64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 设置好pages，等会儿Hexo生成博客push到仓库后，浏览器就可以输入yourname.github.io访问了。

## 配置Hexo

### 安装

这里是[官方中文教程](https://hexo.io/zh-cn/docs/)，照着来就好了～前提条件是要安装git和node.js，用github的话git肯定是装好了，教程都有不赘述了。

### 建博客

安装好Hexo后，终端先到你准备放博客源文件的目录下：

```
hexo blogName

cd blogName

npm install

hexo s
```

现在已经可以在本地预览生成的博客了，在浏览器输入localhost:4000就能看见默认的博客了。

### 关联github仓库

为了能让其他人也能看见你的博客，就要把博客的源文件提交到你的github仓库，打开blog目录下的配置文件_config.yml，修改最底下的几行：

```
deploy:

type: git

repository: git@github.com:yourname/yourname.github.io.git

branch: master
```

yml语法很蛋疼，冒号后面一定要有个空格，之前没打空格被坑过好几次

### 发布博客

```
hexo clean

hexo g

hexo d
```

### Hexo主题

Hexo修改主题相当方便，可以下载别人的主题，自己在主题的基础上再稍作修改，这里推荐两个主题

- [Next](https://github.com/iissnan/hexo-theme-next)，大名鼎鼎的主题，看Star就知道了，主题还自带3种风格，简洁优雅

- [yelee](https://github.com/MOxFIVE/hexo-theme-yelee)，我现在用的主题，喜欢动画效果～

## 自定义域名

> 折腾完前面的步骤，还想继续折腾域名的话就看看吧。

### 购买

首先要先注册一个[阿里云账号](https://www.aliyun.com)，然后就可以去[买域名](https://wanwang.aliyun.com)了。域名的实名认证啥的按照说明自己做就好了。

### 申请DNSPod添加域名管理

> NDSPod的域名解析服务中，提供根据线路进行不同的路由的功能是免费的，这也是为什么选择DNSPod的原因，国外走github.com，国内走coding.net

在阿里云的控制台，找到域名->管理，打开域名控制台，DNS地址修改为f1g1ns1.dnspod.net和f1g1ns2.dnspod.net

![域名DNS修改.png](https://upload-images.jianshu.io/upload_images/2756183-ae907eac3f86495b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在本地的博客目录里的source目录下添加一个名字为CNAME的文件，没有后缀。

![添加CNAME.png](https://upload-images.jianshu.io/upload_images/2756183-5e0448b6ef186910.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在CNAME中添加一行你自己的域名：

![CNAME文件.png](https://upload-images.jianshu.io/upload_images/2756183-435cca9cf4dac131.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之前看教程说CNAME是放在博客根目录，但是hexo d -g发布的时候不会提交根目录文件...

去[DNSpod](https://www.dnspod.cn/)添加域名记录

![添加域名记录.png](https://upload-images.jianshu.io/upload_images/2756183-9c18581306b8fb2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面俩A记录是github的服务器地址，github 会根据每个项目下的 CNAME 生成缓存，从而自动跳转。

最后就是等待ing，我设置的时候等了一个小时就可以访问了。

