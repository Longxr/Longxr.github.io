---
title: NAS 域名访问和 Nginx 反代理
categories:
  - NAS
tags:
  - NAS
  - Nginx
  - docker
date: 2024-03-03 22:51:22
---

<Excerpt in index | 摘要>
有公网 IP 的话可以设置通过域名访问家里的 NAS，另外 NAS 上服务多了端口记不过来，可以给不同的服务设置对应的子域名 <!-- more -->
<The rest of contents | 余下全文>

有了 NAS 之后在家里折腾了一段时间，家里用着不错。如果想要在外面也能访问 NAS 的话，至少还要准备一个域名以及给宽带申请下公网 IP。都有了的话可以参考下述内容开搞~

# 环境准备

域名一个，买个小众点的就行，小众的短域名 10 年也才几百块钱。
公网 IP，可以打电话给运营商，顺便还会上门给你光猫改桥接。

# 环境搭建

## DDNS

先来完成第一个步骤，让 NAS 可以通过域名访问。当买好域名和申请了公网 IP 后，就可以在域名解析的配置里加上 IP 的记录，当访问域名时就会解析到指定的 IP 地址。但是对于普通用户来说申请到的 公网 IP 都是动态的，重启下路由器 IP 就会变，这时候就需要使用 DDNS 服务来定时的更新自己的 IP 到域名解析的记录里了。
![NASDomainAndNginx-2024-03-03-23-38-54](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-03-23-38-54.png)

### 申请 AccessKey

由于我的域名是在阿里云买的，所以这里就用阿里提供的服务了。登录了自己的阿里云账号后点击头像，有个 AccessKey 管理，会提示你使用子账号，跳转到新界面首先根据说明创建一个子账号。
![NASDomainAndNginx-2024-03-03-23-47-49](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-03-23-47-49.png)

阿里的对于每个子账号的权限都可以单独设置，比如我们这里只是用于 DNS 解析，在创建好账号后，权限管理只需要添加 AliyunDNSFullAccess 就行了。
![NASDomainAndNginx-2024-03-03-23-53-44](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-03-23-53-44.png)

在账号的下方认证管理可以创建一个该账号的 AccessKey，创建好后会弹出一个弹窗，上面有 AccessKey ID 和 AccessKey Secret，以及包含这俩的 csv，记得保存好这个 csv，这个弹窗关了之后就看不到了，只能重新再创建了。
![NASDomainAndNginx-2024-03-04-00-00-00](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-00-00.png)

### DDNS-GO 安装

有群晖的话可以配置下[矿神 SDK](https://spk7.imnks.com) 的套件来源，配置好套件来源后，直接搜索安装好 DDNS-GO。
![NASDomainAndNginx-2024-03-04-00-02-25](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-02-25.png)
![NASDomainAndNginx-2024-03-04-00-04-10](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-04-10.png)

### 脚本配置

安装好 DDNS-GO 直接运行并点击打开，就会跳转到其配置页面上，在配置页最上方选好阿里云，并填上 AccessKey ID 和 AccessKey Secret，在下方填好自己的域名并点击最下方的保存就完成了。
**这里如果你申请的域名是 `abc.com`，记得在前面加个星号，填成 `*.abc.com`，因为我们后面是要配置不同服务的子域名的。**
![NASDomainAndNginx-2024-03-04-00-08-21](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-08-21.png)

## 证书添加

现在浏览器访问请求时，如果加了 https 是要校验 ssl 证书的，如果没有的话会提示这个网站有风险。免费证书的话可以在阿里云上申请，搜索【数字证书管理服务】。我之前申请的时候免费的能用一年，不过在写这篇文章的时候登录上去看了下，现在好像不交钱只有 3 个月了，有点坑...

![NASDomainAndNginx-2024-03-04-00-28-57](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-28-57.png)

申请好证书后把证书下载到本地，然后在 安全性里选择证书，新增并导入（第一个证书可以勾选设为默认证书）。在导入界面上选择之前下载的证书文件导入，私钥选择 xxx.key，证书选择 xxx.pem。
![NASDomainAndNginx-2024-03-04-00-35-46](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-35-46.png)

导入完证书后点击设置，将你想要换成自己域名的服务都改成你自己的域名。
![NASDomainAndNginx-2024-03-04-00-44-16](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-44-16.png)

## 端口转发

如果在内网想访问 NAS 上的某个服务，可以通过 `NAS 局域网IP:port` 的方式直接请求。
但是对于外网的访问，即使知道了当前家里的公网 IP，是不能直接访问内网 NAS 机器上的端口的，需要先做下端口转发，使得访问 `公网IP:port` 的请求能转发到 `NAS 局域网IP:port`，现在的路由器都支持该功能，在路由器的配置页设置就行。
**有的路由器有提供 DMZ 功能，这个功能是将局域网内的一个 IP 直接暴露在外网，这个挺危险的，如果有人知道了你的公网 IP 就能不断尝试访问不同的端口，建议不要使用该功能**

我们先转发个 5001 端口，浏览器输入 `https://域名:5001` 就可以看到自己的群晖登录界面。

## 反代理子域名

在完成了上面的内容后，就可以可以通过`https://域名:port`直接访问 NAS 的服务了，此时可以将每个 NAS 服务的端口都通过路由器转发。只是暴露端口每次都要去路由器添加，添加的时候麻烦，用起来也记不住。这时候要反代理上场了，可以只暴露一个端口，不同服务通过子域名来区分，访问连接变成 `https://服务名.域名:18443`。

### 部署 Nginx Proxy Manager

打开群晖的 Container Manager，注册表搜索 `nginx-proxy-manager`，下载 Star 最多的这个即可。
![NASDomainAndNginx-2024-03-04-00-56-22](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-56-22.png)

创建容器指定下自己的端口号和本地路径，由于群晖的 80、443 等端口都是被 Web Station 给占用了，这里需要换成其他自定义的端口如 18080、18443。文件夹映射我的是:

- `/docker/NginxProxyManger/data` => `/data`
- `/docker/NginxProxyManger/letencrypt` => `//etc/letsencrypt`

![NASDomainAndNginx-2024-03-04-00-59-26](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-00-59-26.png)

再勾选下容器的【启用自动重新启动】，配置好后就可以运行容器了。访问的管理端地址就是上面映射的 81 端口，比如我的管理端地址就是：`NAS 局域网IP:18081`。第一次打开这个页面时，默认用户：`admin@example.com`，密码：`changeme`。首次登录后，需要修改用户名、登录邮箱以及登录密码。

### 配置 Nginx Proxy Manager

打开管理端页面配置好邮箱和登录密码后，可以点击 SSL Certificates -> Add SSL Certificate，导入之前创建的，或者申请个新的：
![NASDomainAndNginx-2024-03-04-01-10-45](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-01-10-45.png)

如果是新申请如下，DDNS 的配置上面有介绍过，这里类似，就不再赘述了。
![NASDomainAndNginx-2024-03-04-01-11-41](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-01-11-41.png)

证书添加好后就可以添加自己想要的子域名了，点击 Hosts -> Add Proxy Host:
![NASDomainAndNginx-2024-03-04-01-16-02](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-01-16-02.png)

填好想自己想要的子域名，以及 NAS 服务的端口，比如我有个 test 功能的端口是 23333，就如下图所示：
![NASDomainAndNginx-2024-03-04-01-18-06](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-01-18-06.png)

SSL 配置页记得把所有开关都开下，证书选择上一步添加好的 SSL 证书。
![NASDomainAndNginx-2024-03-04-01-19-55](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/NASDomainAndNginx-2024-03-04-01-19-55.png)

此时再把 Nginx Proxy Manager 的 https 端口 18443（创建容器时自定义的）设置好路由器的端口转发，就可以实现通过 `https://test.域名:18443` 访问 NAS 的服务了。后期有新的服务可以参考同样的形式在 Nginx Proxy Manager 配置 Proxy Host 就能实现，如：`https://test2.域名:18443`

# 参考链接

- [群晖搭建 Nginx Proxy Manager，个人最推荐的反代工具](https://zhuanlan.zhihu.com/p/678250017)
- [群晖添加 SSL 证书，开启 https](https://blog.csdn.net/weixin_42523454/article/details/128100870)
