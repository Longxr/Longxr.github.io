---
title: Hexo使用Travis CI自动化部署
categories:
  - Hexo
tags:
  - Hexo
  - Travis
date: 2017-05-10 13:58:25
---

<Excerpt in index | 摘要> 
通过使用Travis CI自动化部署，换电脑发布文章无需安装Hexo<!-- more -->
<The rest of contents | 余下全文>

## 偷懒是第一生产力
使用Hexo写博客挺简单的，`hexo new "articleName"`在生成的md文件中就可以开始自己的写作了。不过每次都要进行发布的三部曲还是略显麻烦：
```
hexo clean
hexo g
hexo d
```
这时候就到`Travis CI`出场的时候了，通过`Travis CI`可以实现当你将`commit`提交到Github后，会自动帮你进行这3步，你需要的就只是写文章->提交到Github，剩下的都是自动完成。

## Travis CI使用
> **Travis CI**是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在GitHub托管的代码 。

### 登录Travis CI
使用Github账号登录[Travis CI官网](https://travis-ci.org/)，登录后在主界面点击`My Repositories`旁边的"+"号:
![点击加号](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_01.png)

选择你在Github的放博客源码的仓库，打开左侧的开关：

![仓库选择](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_02.png)

![打开开关](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_03.png)

然后点下仓库那里的设置图标，进入设置的界面：

![设置选项](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_04.png)

现在我们就设置好了需要构建的仓库了，Github的仓库指定的分支有commit后就会触发构建，但是目前还没有权限将构建的结果push到Github的仓库，这就需要在Github配置下Access Token了

### 配置Github的Access Token
转到[Github页面](https://github.com/)，在`Setting`界面下选择`Personal access tokens`:

![Personal access tokens](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_05.png)

生成一个新的tokens，设置如下：

![token配置](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_06.png)

> 生成的时候先别急着关页面，配置信息随时可以更改，但是token只在这时候显示一次，页面关了之后找不到的，要的话只能重新生成一个

将复制的token添加到`Travis CI`的环境变量中：

![添加环境变量](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_07.png)

这样`Travis CI`就有了访问Github的权限了

### 配置.travis.yml
我们设置了自动部署，但是具体要怎么部署还没有告诉`Travis CI`，接下来就来设置下相关的配置文件

#### Github的仓库分支
> 如果将源码和生成的博客页面文件放在同一个分支也是可以的，不过每次一提交之后，`Travis CI`会自动化构建然后再push到这个分支。这样会显得比较乱 ，跟默认用hexo三部曲提交到Github差不多，想换台电脑编辑的时候不好找博客源代码，分成两个分支是不错的选择。

由于Github的要求，要作为博客显示的分支必须是master，所以源码就另起了一个分支`blog`。

![另一分支](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_08.png)

#### 添加.travis.yml文件
在blog根目录新建一个.travis.yml文件，添加如下内容：
```
language: node_js   #设置语言
node_js: stable     #设置相应的版本
cache:
    directories:
        - node_modules    #据说可以减少travis构建时间
before_install:
  - npm install -g hexo
  - npm install -g hexo-cli
install:
  - npm install   #安装hexo及插件
before_script:
  - npm install -g mocha
  - git clone --branch master https://github.com/Longxr/Longxr.github.io.git public
script:
  - hexo cl   #清除
  - hexo g   #生成
after_script:
  - cd ./public
  - git init
  - git config user.name "longxr"   #修改成自己的github用户名
  - git config user.email "longxuanren@gmail.com"   #修改成自己的GitHub邮箱
  - git add .
  - git commit -m "update by Travis-CI"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master #GH_token就是在travis中设置的token
branches:
  only:
  - blog  #只监测这个分支，一有动静就开始构建
env:
    global:
        - GH_REF: github.com/Longxr/Longxr.github.io.git
```
修改成你的仓库的配置就行了。设置完毕，就可以新建一篇文章测试下了～以后发文章的流程就是:`写文章->提交到Github`搞定！

## 逼格添加
自动化构建服务还可以在README.md中加上一个构建状态的图标：

![build Stauts](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_09.png)

在自己的README.md中加一行：
`[![Build Status](https://travis-ci.org/Longxr/Longxr.github.io.svg?branch=blog)](https://travis-ci.org/Longxr/Longxr.github.io)`
> branch后设置为监测的分支，链接地址设置为`Travis CI`的仓库地址

也可以在界面点击仓库名后面的状态图标获取链接代码，默认是master分支，记得改成你监测的分支，不然就会显示`unknown`

![获取状态图标链接](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Hexo-Travis-CI_11.png)

> 补充说明: blog分支README.md放在根目录就行，master分支README.md要放在博客目录的/source中

## 参考资料
- [使用 Travis CI 自动更新 GitHub Pages](http://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
- [Travis CI自动部署Hexo博客到Github](https://xin053.github.io/2016/06/05/Travis%20CI自动部署Hexo博客到Github/)
- [Hexo使用TravisCI自动化构建](https://chuyun.github.io/2016/10/06/Hexo-Travis-Automated%20build/)
- [可持续化集成Travis-CI入门教程](http://blog.smallmuou.xyz/git/2016/03/22/可持续化集成Travis-CI入门教程.html)


