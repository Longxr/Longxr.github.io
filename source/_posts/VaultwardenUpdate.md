---
title: Vaultwarden 升级：Bitwarden 客户端兼容问题修复
categories:
  - docker
tags:
  - docker
  - Bitwarden
  - Vaultwarden
date: 2026-08-27 23:40:00
---

<Excerpt in index | 摘要>
Chrome 上的 Bitwarden 插件突然打不开了，一直转圈 loading，网页端也是一样。排查了一圈发现是 Bitwarden 客户端升级到 2026.7.0 后与服务端版本不兼容，把 Vaultwarden 升级到 1.37.0 就好了，顺便把容器改成了 docker-compose 项目管理，以后升级就简单了 <!-- more -->
<The rest of contents | 余下全文>

密码库用了快一年半，一直挺稳定的，最近登录发现网页不会自动填充密码了，插件打开密码库就一直转圈，网页端打开也是一直 loading，登录流程看着都正常，但就是进不去密码库。

## 排查

先看了下服务端的日志，发现登录接口、sync 接口都返回 200，WebSocket 也连接成功了，但前端就是一直在反复轮询 profile 接口，进不了主界面，看着像是前端在 sync 之后卡死了。

![VaultwardenUpdate-2026-08-27-23-41-00](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/VaultwardenUpdate-2026-08-27-23-41-00.png)

再确认了下版本，服务端 Vaultwarden 是 1.36.0，Chrome 插件是 2026.7.0。一查发现官方早就有说明，Bitwarden 客户端升级到 2026.7.0 后与 Vaultwarden 1.37.0 以下版本不兼容，Vaultwarden 官方在 Discussion 里明确写了 "Upgrade to this version when using clients v2026.7.0+"，原来客户端在解密自托管账户数据时 WASM SDK 会崩溃，导致保险库空白或者永远加载不出来。

![VaultwardenUpdate-2026-08-27-23-42-00](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/VaultwardenUpdate-2026-08-27-23-42-00.png)

## 解决

那解决办法就很简单了，把服务端升级到 1.37.0 就行。

### 备份

升级前先备份数据，停止容器后把数据目录复制一份，确认里面有 db.sqlite3 再继续，万一升级出问题还能回滚。

![VaultwardenUpdate-2026-08-27-23-43-00](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/VaultwardenUpdate-2026-08-27-23-43-00.png)

### 改用 docker-compose

之前是用 Container Manager 直接运行创建的容器，每次升级都要手动去记端口、卷、环境变量，然后删容器重建，太容易漏配置了。这次干脆改成 compose 项目的方式，把配置固化到 `docker-compose.yml` 里，以后升级只需要改一下版本号，然后在项目里点下构建就行。

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:1.37.0
    container_name: vaultwarden-server-2
    restart: unless-stopped
    ports:
      - "3332:80"
    volumes:
      - "/volume1/docker/vaultwarden/data:/data"
    environment:
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=false
      - INVITATIONS_ALLOWED=false
      - LOG_FILE=/data/vaultwarden.log
```

数据目录还是原来那个，环境变量也和之前保持一致，所以访问地址、nginx 反代配置都不用动。

### 重建

删掉旧容器，在 Container Manager 里通过项目导入 docker-compose.yml 启动，等镜像拉取完成后，重新打开插件解锁，密码库正常显示了。

![VaultwardenUpdate-2026-08-27-23-44-00](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/VaultwardenUpdate-2026-08-27-23-44-00.png)

## 总结

这次问题本质上是 Bitwarden 客户端与服务端的版本兼容问题，客户端升级到 2026.7.0 后，服务端必须同步升到 1.37.0 才行。另外把容器改成 compose 项目管理后，以后升级就只需要改版本号再重建，不用再担心漏配置了。

## 参考链接

- [v1.37.0 (Upgrade to this version when using clients v2026.7.0+) - Vaultwarden Discussion](https://github.com/dani-garcia/vaultwarden/discussions/7473)
- [Firefox extension 2026.7.0 fails to render vault with self-hosted Vaultwarden - bitwarden/clients](https://github.com/bitwarden/clients/issues/22033)
- [WASM SDK crashes on decrypt for self-hosted (Vaultwarden) accounts - bitwarden/clients](https://github.com/bitwarden/clients/issues/22035)
- [Desktop 2026.7.0 breaks vault display - Vaultwarden Discussion](https://github.com/dani-garcia/vaultwarden/discussions/7505)
- [Bitwarden browser extension / desktop app 2026.7.0 show no entries - Bitwarden Community](https://community.bitwarden.com/t/bitwarden-browser-extension-desktop-app-2026-7-0-show-no-entries/100224)
