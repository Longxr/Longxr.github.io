---
title: Jellyfin 升级：从 nyanmisaka 镜像迁移到官方 12.1
categories:
  - docker
tags:
  - docker
  - Jellyfin
  - NAS
date: 2026-09-20 21:30:00
---

<Excerpt in index | 摘要>
App Store 下载的 Jellyfin 客户端只支持 12 以上的服务端，而我一直在用的 nyanmisaka/jellyfin 镜像停在 2024 年 3 月构建的版本上（10.8 一档），新版客户端直接没法用，只能升级。结果发现官方 12.1 不让一步升上去——旧数据库就是 10.8 时代的，12.1 直接拒绝迁移。中间还因为大肥鱼操作失误把数据库读坏了，又从备份回滚了一次。最后按 10.8 → 10.10.7 → 10.11.11 → 12.1 的路径升级成功 <!-- more -->
<The rest of contents | 余下全文>

Jellyfin 在自己的 NAS 上跑了三年多，用的是 nyanmisaka/jellyfin 这个第三方镜像。它最大的好处是自带比较新的 jellyfin-ffmpeg，群晖的核显硬件转码开箱能用。

这次决定升级的直接原因是客户端：**App Store 下载的 Jellyfin 客户端只支持 12 以上的服务端版本**。而这个镜像的 `latest` 一直停在 2024 年 3 月构建的版本上（对应 Jellyfin 10.8 一档），看了下作者最新镜像没有到12的，等于新版客户端连我的 NAS 上的旧版服务端是不通的。官方镜像早就到 12.x 了，所以干脆切回官方镜像，一次性把版本拉上来。

迁移最大的顾虑不是镜像本身，而是数据。Jellyfin 的媒体库、用户、观看记录全都存在 `/config` 里，而且**数据库里记的是容器内的绝对路径**，所以容器内路径必须一个字符都不改，否则整个库要重建。这次的原则就是：只换镜像，挂载和环境变量尽量照抄。

## 先摸清旧容器的配置

旧容器是用 DSM Container Manager 直接创建的，从容器详情里把配置抄出来：

| 项目 | 值 |
| --- | --- |
| 映像 | `nyanmisaka/jellyfin:latest`（构建于 2024.03.19） |
| 网络 | host / host |
| 存储 | `/volume2/Video` → `/video`、`/volume1/docker/jellyfin/config` → `/config`、`/volume1/docker/jellyfin/cache` → `/cache` |
| 环境变量 | `MALLOC_TRIM_THRESHOLD_=131072`、`LC_ALL`/`LANG`=`en_US.UTF-8`、`LANGUAGE=en_US:en` |
| 内核 | DS920+ |

顺便清掉了几个"当初抄教程带进来、其实没用"的配置：`NVIDIA_DRIVER_CAPABILITIES=compute,video,utility`（这台机器是 Intel 核显，没有 N 卡）、`HEALTHCHECK_URL`（非标准变量，官方镜像不读）。另外旧容器里显式写了 `PATH`，这个也没抄——官方镜像自带 PATH，硬编码反而可能在新版本镜像加了新路径之后把 ffmpeg 找丢。

整理成 `docker-compose.yml`，顺便把已经不用的 AutoBangumi 挂载去掉了：

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:12.1
    container_name: jellyfin-server-1
    network_mode: host
    restart: unless-stopped
    devices:
      - /dev/dri:/dev/dri          # DS920+ 核显直通，QSV/VAAPI 靠它
    environment:
      - MALLOC_TRIM_THRESHOLD_=131072
      - LC_ALL=en_US.UTF-8
      - LANG=en_US.UTF-8
      - LANGUAGE=en_US:en
    volumes:
      - /volume1/docker/jellyfin/config:/config
      - /volume1/docker/jellyfin/cache:/cache
      - /volume2/Video:/video:rw
```

网络保持 host 模式（和旧容器一致），这样 8096 端口和 DLNA 发现的行为都不变，也不用写 `ports:`。

## 第一次尝试：直接上 12.1，被拒

最简单粗暴的做法是把 image 直接改成 `jellyfin/jellyfin:12.1` 启动。容器起来了，但结果很迷惑：**网页全站 503，所有路径都被重定向到 `/startup/logger`**，`/health` 返回 503 Unhealthy，而 `/System/Info/Public` 又能正常返回版本号。

翻日志才看到真正的原因：

```
[FTL] InternalCodeMigration: Error: "Your database does not meet the required standard.
      Only upgrades from server version 10.9.11 or above are supported.
      Please upgrade first to server version 10.10.7 before attempting to upgrade afterwards to 10.11"
[ERR] Migration "20250420193000_MigrateLibraryDbCompatibilityCheck" failed
[FTL] Main: Error while starting server
[ERR] Health check "StartupCheck" with status Unhealthy ... "Server is could not complete startup. Check logs."
```

也就是说 nyanmisaka 那个镜像的库确实是 10.8 档，而 12.1 的迁移例程明确拒绝从 10.9.11 以下直接升。而且这道校验是纯检查、不改库（失败后日志里有 `Attempt to rollback librarydb` / `Attempt to rollback JellyfinDb`）。

官方报错里已经把路指出来了：先升到 10.10.7。

## 升级路径：10.8 → 10.10.7 → 10.11.11 → 12.1

每一跳的做法都一样：改 compose 里的 image、`docker compose up -d`、等日志出现 `Startup complete`、验证、再 `docker compose down` 换下一跳。

其中 10.11 那跳有个大动作：**它把 `library.db` 合并进了 `jellyfin.db`**。数据目录里会多出 `library.db.old`（合并前的原库）、`SQLiteBackups` 目录和 `database.xml`（新版数据库层的配置文件），`jellyfin.db` 从 200 多 KB 涨到 100 多 MB，原来那个 80 MB 的独立 `library.db` 就消失了。这是 10.11 的设计，不是故障。

到 12.1 之后，日志里能看到这轮迁移的全部动作：

```
[INF] Jellyfin.Server.Migrations.JellyfinMigrationService: There are 30 migrations for stage AppInitialisation.
[INF] InternalCodeMigration: Migration "20260910120000_MigrateRatingLevels" was successfully applied
[INF] Main: Migrations have been applied, optimizing the database... This might take a while
[INF] Jellyfin.Database.Providers.Sqlite.SqliteDatabaseProvider: jellyfin.db optimized successfully!
[INF] Main: Startup complete 0:01:12
```

一共 72 条迁移成功执行，之前一直卡住的那道 `MigrateLibraryDbCompatibilityCheck` 这次顺利通过。

## 中途踩的坑：从 SMB 读运行中的数据库

中间为了留个"在线快照"当回滚点，我从 Windows 侧用 SQLite 的 `VACUUM INTO` 去读正在运行的 `jellyfin.db`。结果是这次升级里最严重的一次事故：**两秒之后 Jellyfin 开始狂报数据库损坏**。

```
[ERR] Microsoft.EntityFrameworkCore.Update: An exception occurred in the database while saving changes
      ---> Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 11: 'database disk image is malformed'
[ERR] Emby.Server.Implementations.ScheduledTasks.TaskManager: Error executing Scheduled Task
      at Emby.Server.Implementations.Data.CleanDatabaseScheduledTask.CleanDeadItems
```

几分钟内累积了 86 次，元数据保存和数据库清理任务全部失败。最坑的是**界面上完全看不出来**：网页照样能打开、`/health` 照样返回 200，因为它坏的是写路径（读还有缓存撑着）。判断方法只能是看日志，以及看数据库文件的时间戳——正常写入时 `jellyfin.db` 和 `-wal` 的时间戳会持续推进，而损坏时全都冻结了。

原因是大肥鱼没注意到 **10.11 起 Jellyfin 改用了 WAL 模式**（日志里能看到 `database locking mode has been set to: NoLock`，数据目录里多出 `jellyfin.db-wal` 和 `jellyfin.db-shm`）。WAL 依赖共享内存做锁协调，而 SQLite 官方明确不建议把 WAL 模式的数据放在网络文件系统上——前面几次"在线快照"成功过，纯粹是因为那时候还是旧的事务日志模式，没有 WAL 文件，方法本身一直是错的。

确认损坏的办法是拿一份**停机后的副本**跑 `PRAGMA integrity_check`——它连校验都跑不起来，直接抛 `database disk image is malformed`；而同一批备份里的旧库校验结果都是 `ok`。

恢复过程：停容器 → 用之前验证过的快照还原 `config`（还原后逐文件比对哈希，确认字节一致）→ 重新走升级。数据没丢，媒体文件全程没被动过。

## 在群晖 Container Manager 上踩的两个坑

**1. 删项目会把镜像一起删掉。** 当时为了停止服务在项目里点了"删除"，结果容器和镜像一起没了，10.10.7 的镜像就是这么消失的，后来要回滚还得重新拉。以后只记一条：**停服务用"停止"，改版本用"构建"，明确后续不用再"删除"。**

**2. 别用项目菜单那个"启动"按钮。** 它启动的是已经存在的容器对象，而 image 在容器创建时就固定了。所以会出现"compose 文件改成了新版本，点启动跑起来的还是旧版本"这种迷惑现象——必须用 `docker compose up -d`（或项目里的"构建"）让它按新镜像重建容器。


## 验收标准

实际用的是这几条：

- 日志里出现 `Startup complete`，且**没有任何 `[FTL]`**
- 日志里没有 `does not meet the required standard`（这是库版本不够的信号）
- 日志里没有 `malformed` / `SQLite Error`（这是库损坏的信号）
- 数据库文件的时间戳在持续推进（写入确实在落盘）
- `/health` 返回 200 Healthy，而不是 503
- **观察时间要够长**：那次损坏是在启动完成 3 分半之后才开始的，看一两眼根本发现不了

## 总结

整个升级实际上是三级台阶：11.11 那次数据库合并是最大的一次结构变动，12.1 那次是迁移条数最多的一次。升级完成后 12.1.0 正常运行，媒体库、用户、观看记录都还在。

三条经验后续升级其他容器也注意下：

1. **跨大版本升级要看官方指定的路径**，Jellyfin 会在报错信息里直接告诉你"先升到哪个版本"，别硬跳。
2. **Jellyfin 10.11 之后不要从外部碰数据库文件**（包括只读）。备份只在**停机后**做文件级复制，而且必须把 `-wal` 文件一起复制，否则会丢掉还没落盘的已提交数据；需要校验就复制成副本再校验。
3. **升级前后都用日志和文件时间戳验收**，不要依赖界面能不能打开。

## 参考链接

- [Container | Jellyfin](https://jellyfin.org/docs/general/installation/container/)
- [Jellyfin 12.0 发布说明](https://jellyfin.org/posts/jellyfin-release-12.0/)
- [Release 12.0 · jellyfin/jellyfin](https://github.com/jellyfin/jellyfin/releases/tag/v12.0)
- [Write-Ahead Logging - SQLite](https://www.sqlite.org/wal.html)
- [jellyfin/jellyfin - Docker Hub tags](https://hub.docker.com/r/jellyfin/jellyfin/tags)
