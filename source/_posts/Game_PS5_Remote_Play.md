---
title: PS5 远程玩游戏
categories:
  - Game
tags:
  - PS5
date: 2023-05-21 18:34:30
---

<Excerpt in index | 摘要>
解决下回家躺床上 PS5 吃灰的问题，在公司远程玩 PS5 <!-- more -->
<The rest of contents | 余下全文>

下班回家觉得游戏索然无味，中午在公司又有点无聊，懒得把机器搬到公司去的话，可以试试 PS 的 remote play 功能 (也适用于电视被抢的情况 23333)

# 环境准备

PS5 主机一台
公网 ip，可以打电话给运营商，顺便还会上门给你光猫改桥接。

# 环境搭建

## 端口转发

首先是为了让公司的网络也能访问到家里的 PS5，需要将 remote play 涉及到的端口从路由器转发出去。直接路由器转发端口就可以的前提是你是公网的 ip，并且光猫是桥接模式用的路由器拨号上网。如果路由器没有公网 ip 的话，还要折腾下内网穿透、frp 这些，可以自行搜索处理
PS5 remote play 涉及到的端口有:

- TCP/UDP: 987 (for remote wakeup)
- TCP/UDP: 9295, 9296, 9297, 9303, 9304 (for connection handshake, streaming, and other behavior)

是在 [reddit 找到的端口信息](https://www.reddit.com/r/PS5/comments/kx8mhy/remote_play_from_internet_firewall_ports/)，当然也可以使用路由器的 DMZ 主机直接暴露 PS5 的 ip，但是感觉不安全，还是就转发下端口了

![Game_PS5_remote_play-2023-05-21-19-12-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-12-11.png)

## PS5 设定

PS5 设定比较简单，就是开启下远程游玩的功能，为了方便截图我直接截图 remote play 的窗口了（在家里的 PC 远程到公司的 PC 上，开启 remote play 远程到家里的 PS5 上截图，感觉好蠢哈哈哈哈）

![Game_PS5_remote_play-2023-05-21-19-30-12](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-30-12.png)

![Game_PS5_remote_play-2023-05-21-19-30-44](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-30-44.png)

![Game_PS5_remote_play-2023-05-21-19-31-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-31-11.png)

## remote play 安装

直接去索尼官网下载就行了，[下载链接戳这里](https://remoteplay.dl.playstation.net/remoteplay/lang/en/ps5_win.html)
安装好后需要先登录索尼账号，登录好之后就会搜索你账号下的 PS5 主机

![Game_PS5_remote_play-2023-05-21-19-37-12](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-37-12.png)

搜索到机器后选择你要连接的 PS5 主机，就可以看到画面了

![Game_PS5_remote_play-2023-05-21-19-40-11](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Game_PS5_remote_play-2023-05-21-19-40-11.png)

当然 remote play 也不止 PC 可以用，iOS、Android 也有对应的客户端可以用来远程

# 参考链接

- [Remote Play from Internet Firewall Ports](https://www.reddit.com/r/PS5/comments/kx8mhy/remote_play_from_internet_firewall_ports/)
