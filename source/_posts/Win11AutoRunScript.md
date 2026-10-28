---
title: Win11 定时开机自动跑脚本：从 BIOS RTC 到 VSCode 终端插件
categories:
  - 操作技巧
tags:
  - Windows
  - VSCode
  - BIOS
  - 任务计划程序
  - 自动化
date: 2026-09-24 22:50:00
---

<Excerpt in index | 摘要>
自动化运行游戏每天开机也嫌麻烦了，让电脑自己开机自己玩(￣▽￣)" <!-- more -->
<The rest of contents | 余下全文>

## 需求

要的东西很具体：

- 每天早上到点，机器自己自动开机
- 自动打开 VSCode，运行我自动化运行游戏的脚本，通过 VSCode 是正好可以看运行状态和终端日志
- 全程不要人干预，因为那个点正好是起床时间，顺便也可以当个闹钟。

主要步骤拆解为：BIOS 设定时开机、开机跳过锁屏、任务计划设开机启动 VSCode、插件设置自动跑命令。4个步骤串行执行，有一环卡住的话就要起床手动操作了。

## 第一环：定时开机

Win11 里没有"定时开机"这个开关。任务计划程序里那个"唤醒计算机运行此任务"也帮不上忙：它依赖 ACPI 的唤醒定时器，只能把机器从睡眠/休眠里叫醒。

关机状态要上电，只能让主板来做。进 BIOS，在电源管理里找 `RTC Wake` / `Resume By RTC` / `Auto On Time` 这一类名字，不同厂商叫法不一样。

这块板子上有几个容易选错的项：

| 选项 | 含义 | 怎么选 |
| --- | --- | --- |
| RTC Wake | 到点给电，真正的定时开机 | 启用 |
| Wake by LAN | 收到网络包才开机 | 关掉 |
| Fixed Time | 每天固定时刻 | 选这个 |
| Dynamic Time | 相对时间/延迟开机 | 别选 |

`Date` 填 0 表示每天，再填时、分、秒。`Fixed Time` 和 `Dynamic Time` 不是两个开关，是同一个闹钟的两种计时方式，后者按相对时间算，拿来设定时开机只会把时间搞乱。

`Wake by LAN` 和 `RTC Wake` 不建议同时开：两者都启用时，有些主板会让 RTC 的设定被网络事件盖掉；反过来，局域网里的广播包也可能在你不想要的时候把机器叫起来。真要远程开机，更稳的是留 RTC Wake，再用"通电自启"（AC Power Loss → Power On）配一个能远程控制的插座。

时间不用换算时区，直接填北京时间。Windows 默认是**把本地时间写进 RTC** 的（这一点和 Linux 默认用 UTC 相反），所以 BIOS 里看到的就是系统里的时间。要保证的只有一件事：系统时间本身是准的。

还有一条硬约束：**插座不能断电**。定时开机靠的是主板上那个一直在走的时钟，拔了电源或者关掉插排，计时就没了。

## 第二环：开机之后把 VSCode 拉起来

任务计划程序（`taskschd.msc`）里建任务，触发器选"登录时"，操作填：

```
程序或脚本: C:\Users\<user>\AppData\Local\Programs\Microsoft VS Code\Code.exe
添加参数:   "D:\workspace\GithubCode\python-data-process"
起始于:     D:\workspace\GithubCode\python-data-process
```

这里有个坑人的选项：**"不管用户是否登录都要运行"**。

它听起来正是"锁屏也能跑"的答案，实际上勾上之后，VSCode 的窗口根本不会出现在桌面上。这个选项把任务放进非交互会话（session 0）执行，进程确实起来了，但没有桌面可画——它是给服务型任务用的，不是给"帮我把编辑器打开"用的。**想看见窗口只有一个前提：已经有人登录，而且桌面会话没锁。** 


## 第三环：让 VSCode 一打开就执行命令

### 装错的那个插件

我一开始装的是 `saurabh.terminal-manager` 1.0.6。按教程在项目里建了 `.vscode/terminal-manager.json`，写了 `runAutomatically: true`，重载窗口，终端一动不动。

后来才搞清楚它是什么：一个**终端配置切换器**——预定义几套 shell（CMD / PowerShell / Bash），在侧边栏点一下切过去。配置在 `%APPDATA%\Code\User\globalStorage\saurabh.terminal-manager\terminals.json`，字段是 `shellPath` / `shellArgs`，既没有 `commands` 也没有 `runAutomatically`。项目里那个 JSON 它根本不读。


### 能用的那个

真正能做"打开项目就执行命令"的是 `fabiospampinato.vscode-terminals`，商店显示名也叫 **Terminals Manager**。别搜名字，直接按 ID 装：

```
ext install fabiospampinato.vscode-terminals
```

它读项目根目录下的 `.vscode/terminals.json`：

```json
{
  "autorun": true,
  "terminals": [
    {
      "name": "AutoGame",
      "cwd": "D:/workspace/GithubCode/python-data-process",
      "commands": [
        "cd auto_game",
        "python ./main.py --all"
      ],
      "focus": true
    }
  ]
}
```

几个实际用到的点：

- `autorun` 是全局开关，默认 `false`，不写就什么都不发生；也可以按终端单独设。触发时机是"VSCode 启动"和"项目被加入工作区"。
- `cwd` 是终端的初始目录，`commands` 会按顺序敲进终端、每条都回车。
- 文件按 **JSONC** 解析，注释和尾逗号都能留。
- 路径用正斜杠，省掉转义的问题。
- `focus: true` 把焦点给这个终端，方便看输出；不想抢焦点可以换成 `open: true`。


## 第四环：锁屏，以及自动登录

定时开机不会顺便帮你解锁。机器起来了，停在锁屏界面，而第三环的插件要往终端里写命令，它需要一个已登录、没锁的桌面会话。

| 方案 | 代价 | 适用 |
| --- | --- | --- |
| 自动登录 | 开机跳过锁屏，碰到机器的人直接进桌面 | 家里的专用台式机 |
| "不管用户是否登录都要运行" | 进程能跑，GUI 不可见 | 纯后台任务 |
| 不做 GUI，改无头执行 | 看不到实时输出 | 真正要无人值守的任务 |

实现操作踩了下面几个坑：

### 注册表三件套，填 PIN 不行

Winlogon 下的三件套：

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
  AutoAdminLogon   REG_SZ  1
  DefaultUserName  REG_SZ  xxx@outlook.com
  DefaultPassword  REG_SZ  <密码>
```

照着"PIN 也能填"的教程做，重启后卡在登录界面：

> 目前无法连接，请检查网络后再试一次。

原因不复杂：**自动登录走的是 LSA 里的账户密码凭证，PIN 不是密码。** PIN 是 Windows Hello 用 TPM 在本地校验的便捷凭证，没办法替账户密码去做身份验证。系统拿着一个"看起来像密码"的 PIN 去微软账户那边验证，开机那一瞬间网络还没就绪，就报"无法连接"。同一个账户、同一个 PIN，手动敲能进、自动登录进不去，差别就在这里。

两条出路：`DefaultPassword` 填微软账户的在线密码（代价是密码明文躺在注册表里），或者换 Sysinternals 的 Autologon。

### Autologon

界面就三个框：用户名、域（本地机器留空）、密码。它和手改注册表做的是同一件事，但密码作为 **LSA secret** 加密存放，不是明文；这也意味着有管理员权限的人依然能解出来，它只防"顺手看见"。

三个细节：

- 它不校验填进去的凭证，填错了下次开机才知道；
- 开机时按住 `Shift` 可以临时跳过这一次自动登录，算是个逃生口；
- 微软账户登录的话，用户名填邮箱，密码填账户在线密码，不是 PIN。

自动登录的安全边界很直白：**谁碰到这台机器，谁就是这台机器的主人。** 这台是放在家里的台式工作机，才敢这么干；笔记本、要带出门的机器不要开。手动 `Win+L` 锁屏依然有效，自动登录只影响开机那一次。


## 总结

"串联型"需求，单看每一步都是对的，但没有人替你验证"这一步到底有没有生效"。真正省时间的做法是先把链路上的四件事单独跑通，再全部串行执行完成一个完整需求。

## 参考链接

- [Autologon - Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/autologon)
- [Protecting the Automatic Logon Password](https://learn.microsoft.com/en-us/windows/win32/secauthn/protecting-the-automatic-logon-password)
- [fabiospampinato/vscode-terminals](https://github.com/fabiospampinato/vscode-terminals)
- [Terminals Manager - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=fabiospampinato.vscode-terminals)
- [Profiles in Visual Studio Code](https://code.visualstudio.com/docs/configure/profiles)
- [Task Scheduler - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page)
