---
title: 远程办公内网搭建
categories:
    - 环境搭建
tags:
    - zerotier
    - vscode
    - 内网穿透
date: 2020-03-21 09:41:27
---

<Excerpt in index | 摘要>
通过 zerotier 搭建远程办公的内网，操作简单，关键不花钱 2333 <!-- more -->
<The rest of contents | 余下全文>

## 起因

平时如果需要在公司访问家里的电脑一般都是用的 teamviewer，免费版一般都够用，没啥问题。可是最近也不知道是不是因为远程办公的人太多了，teamviewer 经常显示我要连的计算机离线，没关机的电脑要怎么离线嘛*(:з)∠)*于是我就重新找了下有没有其他好用的内网穿透的工具。

## 对比

内网穿透说白了就是内网机器没有对外的 IP，从外部不能直接访问，就需要有个站在外面的中间人转发下消息。看了下现有的工具，应该是分为两类：

1. 有第三方中心服务器的，teamviewer、向日葵、zerotier...
2. 需要有公网 IP 的服务器的，frp、SoftEther...

我的需求是自己能访问就行，也不需要像服务一样暴露给其他人，懒得再另外搞个服务器了，果断选择了第一类。teamviewer 由于会抽风跳过; 向日葵搜了下收费版风评好像也没比 teamviewer 的免费版好，也跳过; 所以选择了 zerotier。

## zerotier 原理

zerotier 使用时会创建一个虚拟局域网（VLAN），所有加入这个网络的机器可以像是在一个局域网一样互相访问。只需要管理员去官网注册一个账号并创建网络，其他人[安装 zerotier 的客户端](https://www.zerotier.com/download/)并加入对应 VLAN 即可。

![Remote-Access-Server-2020-3-20-22-19-51.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-22-19-51.png)

## 创建 VLAN

### 注册一个账号

[官网链接](https://my.zerotier.com/)，注册略过

### 创建一个新的网络

点击网站上方的 NETWORK，然后再点击蓝色的`Create a Network`，就会自动创建一个网络，网络会显示个网络 ID 在蓝色按钮下面

![Remote-Access-Server-2020-3-20-22-35-10.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-22-35-10.png)

### 网络具体配置

点击上一步创建的网络，可以进入网络配置。

1. 下图的权限控制，`PRIVATE`表示加入网络的客户端需要你在下面的设备列表勾选了才能进入这个 VLAN；`PUBLIC`表示任何加入的客户端都能直接进入。
2. `IPv4 Auto-Assign`中随便选一个你想要的 VLAN 的网段
3. 底下的那个显示设备 MAC 地址、VLAN IP 和公网 IP 的列表是加入这个网络的设备，`AUTH`勾选了表示允许加入了这个网络，`Managed IPs` 可以给每个设备一个想要的 IP
4. 点击设备名称旁边的小扳手，`Allow Ethernet Bridging`网络中的设备需要互相通信的话要勾选; `Do Not Auto-Assign IPs`我是勾选了不自动分配，自己给每个设备固定设置 IP

![Remote-Access-Server-2020-3-20-22-32-45.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-22-32-45.png)
![Remote-Access-Server-2020-3-20-22-43-38.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-22-43-38.png)

## 客户端安装

[安装 zerotier 的客户端](https://www.zerotier.com/download/)，各个平台按照说明安装即可，加入网络后如果网络设置的 PRIVATE，需要管理员在配置页面允许加入下，配置说明可以看上面的说明

### windows

window 最简单，下载 msi 直接安装就行了。装好后在通知栏有个小图标，右键 Join Network，填上上面创建的网络的 ID 就加入了
![Remote-Access-Server-2020-3-20-22-58-58.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-22-58-58.png)

### centos

#### 添加 yum 源

`vim /etc/yum.repos.d/zerotier.repo`

```
[zerotier]
name=ZeroTier, Inc. RPM Release Repository
baseurl=http://download.zerotier.com/redhat/el/$releasever
enabled=1
gpgcheck=0
```

#### 命令行操作（root)

```
yum clean all
yum install zerotier-one                     #安装
zerotier-one -d                              #启动
zerotier-cli status                          #查看状态
zerotier-cli join <Network ID>               #加入网络
systemctl start zerotier-one.service
systemctl enable zerotier-one.service        #开机自启
```

## VSCode Remote SSH

同一个局域网的 windows 直接远程桌面就可以愉快的玩耍了， windows 用 xshell 等工具连接 linux 服务器操作的话还是有点蛋疼，尤其是需要修改代码啥的。
`Remote SSH`是 VSCode 远程开发包中的一个扩展，在目标 linux 服务器配置好 ssh 后，可以在本地直接用 VSCode 远程连接写代码，用起来超方便。

![Remote-Access-Server-2020-3-20-23-33-51.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-23-33-51.png)

### ssh 配置

#### 密钥生成

1. 生成 ssh 密钥，`ssh-keygen -t rsa -C"你的邮箱"`
2. 编辑`~/.ssh/config`（windows 在 C:\Users\xxx\.ssh），配置好服务器 User(用户名)、HostName（服务器 IP）、IdentityFile，
3. 终端敲命令, `name@server-ip`根据你的服务器和用户名填写

```
SET REMOTEHOST=name@server-ip
scp %USERPROFILE%\.ssh\id_rsa.pub %REMOTEHOST%:~/tmp.pub
ssh %REMOTEHOST% "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat ~/tmp.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm -f ~/tmp.pub"
```

#### 错误处理

有时候直接用 VSCode 连接 linux 服务器会报错，如果错误是 wget 下载文件错误的话，在 VSCode 输出日志上面应该有一串哈希值的 commit-id，对应 vscode 某个 release 版本。直接用本地电脑访问  https://update.code.visualstudio.com/commit:<commit-id>/server-linux-x64/stable 先下载下来，下载后将`vscode-server-linux-x64.tar.gz`文件解压到`~/.vscode-server/bin/<commit-id>`，后面再用 VSCode 远程应该就可以了

## 用后感

-   访问同一个 VLAN 的机器速度还是很快的，windows 的话直接远程桌面和 teamviewer 比基本上是没区别了，而且还稳定。
    ![Remote-Access-Server-2020-3-20-23-9-36.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Remote-Access-Server-2020-3-20-23-9-36.png)

-   传输文件比较弱，通过 ssh 的 scp 传输文件的时候，最开始 10 秒还可以 2M/s,后面就缩水成 kb/s 了，不过只要网连着，找个网盘中转下就行
-   对于一些不方便安装客户端的设备，可以通过已经加入网络的客户端，做一下端口映射，相关工具比如[gost](https://github.com/ginuerzh/gost)，有需要的自己去看看吧
