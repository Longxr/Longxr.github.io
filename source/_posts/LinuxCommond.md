---
title: Linux 常用命令记录
categories:
  - Linux
tags:
  - Linux command
date: 2023-07-04 23:43:15
---

<Excerpt in index | 摘要>
记录常用的终端命令 <!-- more -->
<The rest of contents | 余下全文>

# 服务器信息

## 内核版本

终端显示内核命令，`uname -r`

[内核发布地址](https://www.kernel.org/)，主版本号、次版本号(奇数开发版、偶数稳定版)、末版本号

## 发行版本

终端显示发行版本，`lsb_release -a`

1. RedHat，技术支持收费版，软件更新慢，稳得一笔
2. Fedora，软件新，稳定性一般
3. CentOS，软件较新又稳定，小公司服务器首选
4. Ubuntu，桌面版选择
5. Debian，桌面版选择

# 软件包安装

## 常用安装源替换

1. [阿里](https://developer.aliyun.com/mirror)
2. [网易](http://mirrors.163.com/)
3. [腾讯](https://mirrors.cloud.tencent.com/)

## rpm 安装

### 特点

需要软件的 rpm 安装包，并且有依赖的安装包需要自己先安装依赖。
rpm 包格式：name-1.2.33-el7.x86_64.rpm，el7 表示 centos7、RedHat7...

### rpm 参数

- -q xxx 查询软件包，-qa | more 查询所有安装的包
- -i 安装软件包
- -e 卸载软件包

## yum 安装

### 特点

在线安装，安装软件时会自动处理依赖问题。

### yum 参数

- install 安装软件包
- remove 卸载软件包
- list | grouplist 查看软件包
- update 升级软件包
- yum 被锁定时，rm -f /var/run/yum.pid

### 更换 yum 源

去[阿里云镜像站](https://developer.aliyun.com/mirror) 复制替换源的命令
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache

## 源代码安装

### 特点

对于一些在当前系统没有发布安装包，或是需要自己修改源码的软件，可以通过自己编译来安装。

```
//下载，解压到指定xxx目录
./configure --prefix=/usr/local/xxx
make -j2
make install
```

# 服务器交互

## centos 访问 windows 共享文件夹

```
sudo mkdir /mnt/windows
sudo mount -t cifs -o username=smb,password=shared //192.168.31.31/shared /mnt/windows
cp -r /mnt/windows/test/ ./
sudo chmod -R 777 ./test
```

## centos ssh 登录

1. 生成 ssh 密钥，ssh-keygen -t rsa -C"github 邮箱"
2. 编辑.ssh/config，配置好服务器 User、host、IdentityFile
3. 终端操作

```
SET REMOTEHOST=name@server-ip
scp %USERPROFILE%\.ssh\id_rsa.pub %REMOTEHOST%:~/tmp.pub
ssh %REMOTEHOST% "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat ~/tmp.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm -f ~/tmp.pub"

//上面要是连接还需要密码的话就重启下服务器ssh服务
service sshd restart
```

## vscode remote-ssh

### 群晖 remote-ssh

```shell
cd /etc/ssh
sudo vim sshd_config
# 修改 AllowTcpForwarding: no 为 AllowTcpForwarding : yes
:wq #vim 改完保存文件
sudo synosystemctl restart sshd.service # 重启群晖 ssh 服务
```

### centos 下载报错

连接如果出现错误，错误日志为 wget 下载失败，在上面应该有一串哈希值的 commit-id，对应 vscode 某个 release 版本。直接用本地电脑访问  https://update.code.visualstudio.com/commit:<commit-id>/server-linux-x64/stable
下载后将`vscode-server-linux-x64.tar.gz`文件解压到`~/.vscode-server/bin/<commit-id>`，
scp vscode-server-linux-x64.tar.gz user@ip:/home/longxr/.vscode-server/bin/<commit-id>/
tar zxf vscode-server-linux-x64.tar.gz -C .

### centos 登录出错

1. 换机器登录服务器可能导致服务端装了不同的 vscode-server 导致不断重连让选择平台，可以先用 xshell 登录删除相关目录：`sudo rm -rf /home/longxr/.vscode-server`
2. 连接失败提示命名管道不存在，删除本机 ssh 目录下的`known_hosts`文件

# 常用目录

- / 根目录
- /bin 命令保存目录（普通用户权限）
- /sbin 命令保存目录（root 权限）
- /boot 启动目录，包含启动相关文件，和开机有关
- /dev 设备文件保存目录
- /etc 配置文件保存目录，类似 windows 注册表
- /home 普通用户家目录
- /lib 系统库保存目录
- /mnt 系统挂载目录
- /media 挂载目录（常用于光盘挂载）
- /root 超级用户家目录
- /tmp 临时目录
- /proc 直接写入内存的
- /sys 直接写入内存的
- /usr 系统软件资源目录
- /var 系统相关文档内容

# 命令

## 常用命令

```
//查看目录下内容
ls -l            //长格式显示文件，-lh以便于人识别的方式显示（文件大小），-lhS列表按照文件由大到小显示
ls -a           //显示隐藏文件
ls -r           //逆序显示
ls -t           //按时间顺序显示
ls -R           //递归显示，可以查看多级目录

//切换目录
cd floder
cd ./floder

//新建目录
mkdir a b c
mkdir -p a/b/c  //新建多级目录

//删除
rmdir a         //只能删除空目录
rm -rf           //删除非空目录
rm -rf /*       //删库跑路

//复制
cp  /src /dst       //复制文件
cp -r /src /dst     //复制目录
cp -v /src /dst     //终端显示从源到目标的箭头
cp -p /src /dst     //复制并保留文件用户、时间、权限
cp -a /src /dst     //复制并保留文件所有属性，等于-dpR
cp -Rf /src/*   /dst/   //将src目录下的所有东西拷到temp下而不拷贝src目录本身

//移动
mv /src /dst        //重命名、改名

//修改目录下所有文件权限
sudo chmod -R 777 /test
//修改目录下所有文件所属用户
sudo chown -R myuser /path/to/folder

//查看文本文件
cat a.txt             //查看文件内容
head -5 a.txt       //查看文件前5行
tail -5 a.txt        //查看文件末尾5行
wc -l a.txt         //输出文件行数
cat app.log | grep -30 ‘keyword’ //查询关键字前后30行

//打包压缩
tar cf test.tar src/          //只打包，大
tar czf test.tar.gz src/      //gzip压缩，快
tar cjf test.tar.bz2 src/     //bzip2压缩，慢
zip -r test.zip src/          //zip压缩

//解压缩
tar xf test.tar -C dst/        //tar解包
tar zxf test.tar.gz -C dst/   //gzip解包，.tgz同
tar jxf test.tar.bz2 -C dst/  //bzip2解包，.tbz2同
unzip -o test.zip -d dst/                  //uzip解压缩
```

## 特殊技巧

```
//权限被据绝的时候，会将上一条命令加上sudo输出
sudo!!

//切换应用到后台
Ctrl+Z  //暂停应用程序
...        //处理其他事情
fg      //切换到之前的应用

//ssh登出后仍然运行命令
nohup wget http://xxxx
```

## 命令行软件

- htop 进程管理
- range 文件浏览器

# 配置文件/etc

/etc/passwd   　　用户数据库，每列的内容分别是用户名、是否需要密码、uid、组 id、家目录、登录的命令解释器（sbin/nologin 表示不能登录）
/etc/group   　　 组数据，每列的内容分别是组名、是否需要密码、组 id、组员

## 给普通用户管理员权限

```
su root
visudo          //vim /etc/sudoers

//找到root All=(ALL) ALL 这一行
按yy复制行，回车按p粘贴行
命令模式 :wq!保存
```

## sudo 命令输入一次后，不关闭终端无需再输入密码

```
sudo vim /etc/sudoers
//找到Defaults env_reset，改成下面的：
Defaults env_reset , timestamp_timeout=-1
```

# 文件权限(root 用户不受限制)

目录：r 是可查看文件，w 是修改文件，x 是进入文件夹
用户和组权限不同时，以用户权限为准
chmod u g o a +/-/= r/w/x，给用户、组、其他用户添加、删除、设置权限
chmod 744 设置权限，r 代表 4，w 代表 2，x 代表 1，相加设置一个用户权限

## boot 空间不足

一般是系统升级了之后，旧的内核文件还在，多了就满了。可以删除不需要的旧内核文件。

```
su
uname -r                                             //查看当前内核版本
rpm -q kernel                                       //列出所有内核文件
rpm -e kernel-3.10.0-957.el7.x86_64          //删除旧版本
yum remove kernel-3.10.0-957.el7.x86_64   //移除旧版本的相关包
df -lh                                                  //查看磁盘占用
```

# 网络

## 网卡名称

- eno1 板载网卡
- ens33 PCI-E 网卡
- enp0s3 无法获取物理信息的 PCI-E 网卡
- eth0 以上都不匹配

### 网卡显示 eth0 的方法

修改/etc/default/grub，swap rhgb quiet 后面增加 biosdevname=0 net.ifnames=0
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot

## 网络相关命令

```
//修改ip地址
ifconfig eth0 <ip> netmask 255.255.255.0
//重启网卡（改ip要重启网卡才生效）
systemctl restart network


//查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports
//端口开关
firewall-cmd --zone=public --add-port=3306/tcp --permanent   // 开放3306端口
firewall-cmd --zone=public --remove-port=3306/tcp --permanent  //关闭3306端口
firewall-cmd --reload   // 配置立即生效

//查看网络连接状态
mii-tool eth0   //虚拟机看不了

//网络故障排除
ping <ip>
traceroute -w 1 //查看路由跳转，查询丢包原因，更详细可以使用mtr看
nslookup www.baidu.com  //查看域名对应的ip
telnet <ip> <port>  //查看服务端口是否启用
tcpdump -i any -n host <ip> -w /tmp/filename //捕获对应ip的tcp数据包
```

# 进程管理

## 常用命令

```
ps                          //查看当前运行的进程
ps -e | more            //查看所有进程
ps -ef | grep xxx        //查找xxx进程，加了f参数第三列显示父进程ID
pstree | more           //树状显示进程
top                        //查看进程详细运行状态，类似windows资源管理器
kill -s 9 <pid>         //根据pid杀进程
```
