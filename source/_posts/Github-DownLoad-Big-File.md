---
title: Github仓库过大、文件过多下载方案
categories:
  - 环境搭建
tags:
  - git
  - VPS
date: 2018-10-28 12:58:12
---

<Excerpt in index | 摘要> 
当项目的文件过大或者小文件数量过多的时候，直接从github下载经常因为网络问题中断，而且还没个恢复下载的选项只能重新开始相当坑了....<!-- more -->
<The rest of contents | 余下全文>

### 方案一 码云中转
1. 先fork需要下载的仓库
2. 在[码云](https://gitee.com)中绑定自己的[Github](https://github.com)账号
3. 点击码云页面右上角的"+"==>从Github导入项目，就可以同步fork的仓库了
![导入项目](https://upload-images.jianshu.io/upload_images/2756183-008f666a511e238a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 点击进入项目==>右侧的克隆/下载就能下载打包好的zip了

PS: 码云的压缩率有点扯，整个仓库实际有1.3G，压缩的zip是718M.....

### 方案二
1. 搞个服务器VPS。推荐[Vultr](https://www.vultr.com)，按小时收费，不用了可以直接删除，删除后不扣钱，就算连开一个月也才5美刀（最近好像有3.5美刀/月的套餐了）
2. 登录VPS终端 (可用xshell) ，在VPS的终端上：`git clone ....`
3. 打包整个项目 ：`zip -r name.zip ./*` (name=压缩文件名 ./* 压缩当前目录所有文件)
4. 通过一个FTP工具 , 我用的[FileZilla](https://filezilla-project.org), 连上你的服务器就可以下载对应的压缩包了

PS: VPS上clone是真的快，30M/s，1.3G压缩完是150M左右，下载150M的文件就很快了



