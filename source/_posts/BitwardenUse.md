---
title: 密码库迁移到 Bitwarden
categories:
  - docker
tags:
  - docker
  - Bitwarden
date: 2025-02-22 21:40:13
---

<Excerpt in index | 摘要>
之前多端的密码库使用的是 LastPass，在电脑上使用免费没什么问题，直到有次想不起密码在手机上用时才发现有平台限制。然后看了下支持自己搭建密码库的方案，决定就是 Bitwarden 了。 <!-- more -->
<The rest of contents | 余下全文>

## LastPass 的限制

LastPass 如果只在同一个平台使用的话是没有什么问题的，但如果换了平台就会有如下提示，想要用的话，得加钱 ~

![BitwardenUse-2025-02-22-21-45-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-45-11.png)

加钱是不可能加的，看了下 Bitwarden 跨平台各端都支持，也支持自托管服务器，把密码库放到自己的服务器上，就决定是你了!

## Bitwarden 安装

我是用的群晖，可以通过 Container Manager 来安装 docker 镜像。在注册表搜索 `vaultwarden/server` 就可以下载了，下载了较新的 1.33.2 版本 （没有直接下载 latest 是因为跑个几年后自己都不记得 latest 是哪个版本了哈哈哈）

下载后运行镜像，需要设置下环境和存储空间位置两部分内容。

### 环境设置

环境设置首次启动时可以用默认的就行，后续步骤先创建好自己的账号后，可以选择禁止创建账号，就自己一个人使用。下面三个新增的变量功能分别是：

- SIGNUPS_ALLOWED，允许注册，false 就是禁止注册，第一次启动先别加，不然自己都注册不了了
- INVITATIONS_ALLOWED，允许邀请，false 就是禁止邀请
- 日志文件位置，/data/bitwarden.log

![BitwardenUse-2025-02-22-21-48-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-48-11.png)

### 存储空间位置

容器内部数据都在 `/data`，指定到本地一个文件夹即可，比如 `docker/vaultwarden/data`

![BitwardenUse-2025-02-22-21-45-14](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-45-14.png)

容器的 80 端口映射一个可用端口就行，这里映射到 3332，配置好后下一步直接运行。由于 Bitwarden 的安全限制必须使用 https 访问，所以直接通过链接 `http://NAS_ip:3332` 是不行的，需要反代理设置下，参考之前的文章[设置反代理](https://longxuan.ren/2024/03/03/NASDomainAndNginx/)。

## Bitwarden 使用

设置好后通过 https 访问代理地址就可以看到首页了，注册账号并登录可以看到当前账号的管理页面。

![BitwardenUse-2025-02-22-21-45-40](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-45-40.png)

### 数据迁移

首先要做的就是先把 LastPass 的密码数据给迁移过来，先登录 LastPass，在 Advanced Options 里，选择 Export，验证后会将所有密码导出到一个 csv 文件中。

![BitwardenUse-2025-02-22-21-46-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-46-11.png)

然后回到 Bitwarden 这边，选择导入数据，文件格式选择 LastPass，文件选择上一步导出的 csv 导入数据。

![BitwardenUse-2025-02-22-21-46-46](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-46-46.png)

### 自托管地址设置

服务搭建好了，密码也迁移完成，最后一步就是在不同的平台、浏览器上使用了。客户端和浏览器扩展可以在[官网直接下载](https://bitwarden.com/download/)。下载并安装好后，在登录的地方，可以选择使用自托管环境，并将服务器 URL 指定为之前反代理设置的链接，然后就可以在各个平台愉快的使用了。

![BitwardenUse-2025-02-22-21-47-21](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/BitwardenUse-2025-02-22-21-47-21.png)

## 参考链接

- [登录信息就该自己掌握：基于私有云的 Bitwarden 迁移指南](https://sspai.com/post/61976/)
- [群辉 7.2 Container Manager 套件搭建 Bitwarden 开源密码库](https://zhuanlan.zhihu.com/p/682424886)
