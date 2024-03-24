---
title: docker 常用命令
categories:
  - docker
tags:
  - docker
date: 2024-03-25 22:41:11
---

<Excerpt in index | 摘要>
记录下常用的 docker 命令 <!-- more -->
<The rest of contents | 余下全文>

## 底层技术支持

- Namespaces: 做隔离 pid，net，ipc，mnt，uts
- Control groups: 做资源限制
- Union file systems: Container 和 image 的分层

## Docker 使用

### 常用命令

```
sudo systemctl start docker                               //启动docker
docker pull ${CONTAINER NAME}                             //拉取镜像
docker images                                             //查看本地所有镜像
docker ps                                                 //查看所有正在运行的容器，加-q返回id
docker ps -a                                              //查看所有容器，加-q返回id
docker rmi ${IMAGE NAME/ID}                               //删除镜像
docker rm ${CONTAINER NAME/ID}                            //删除容器
docker save ${IMAGE NAME} > ${FILE NAME}.tar              //将镜像保存成文件
docker load < ${FILE NAME}.tar                            //从文件加载镜像
docker start ${CONTAINER NAME/ID}                         //运行一个以前运行过的容器
docker stop ${CONTAINER NAME/ID}                          //停止一个正在运行的容器
docker logs ${CONTAINER NAME/ID}                          //显示运行容器的日志
docker run...                                             //运行一个容器
    --name ${container name}                              //设置容器名称
    -it                                                   //启动交互式容器, 默认打开一个终端，后面还可以写终端命令，会替换Dockerfile的CMD
    -p ${host port}:${container port}                     //指定端口映射
    -P                                                    //随机端口映射，将容器暴露的端口(EXPOSE) 随机映射到宿主机的一个端口，访问宿主机端口就相当于访问容器端口
    -e ${env name}=${env value}                           //添加环境变量
    -d                                                    //后台运行，守护式容器
    -v ${host folder path}:${container folder path}       //将主机目录挂在到容器内，类似软链接文件是同步的，$(pwd)代表当前目录
```

### 高级命令

```
# Advance use
docker inspect ${CONTAINER NAME/ID}                       //查看容器信息
docker ps -f "status=exited"                              //显示所有退出的容器
docker ps -a -q                                           //显示所有容器id
docker ps -f "status=exited" -q                           //显示所有退出容器的id
docker restart $(docker ps -q)                            //重启所有正在运行的容器
docker stop $(docker ps -a -q)                            //停止所有容器
docker rm $(docker ps -aq)                                //删除所有容器
docker rm $(docker ps -f "status=exited" -q)              //删除所有退出的容器
docker rm $(docker stop $(docker ps -a -q))               //停止并删除所有容器
docker start $(docker ps -a -q)                           //启动所有容器
docker rmi $(docker images -a -q)                         //删除所有镜像
docker exec -it ${CONTAINER NAME/ID} /bin/bash            //进入容器内
docker exec -it ${CONTAINER NAME/ID} ping ${CONTAINER NAME/ID} //一个容器ping另外一个容器
docker exec -it ${CONTAINER NAME/ID} ip a                 //查看容器网络接口
docker top ${CONTAINER NAME/ID}                           //显示一个容器的top信息
docker stats                                              //显示容器统计信息(正在运行)
docker stats -a                                           //显示所有容器的统计信息(包括没有运行的)
docker stats -a --no-stream                               //显示所有容器的统计信息(包括没有运行的) ，只显示一次
docker stats --no-stream | sort -k8 -h                    //统计容器信息并以使用流量作为倒序
docker system
docker system df                                          //显示硬盘占用
docker system events                                      //显示容器的实时事件
docker system info                                        //显示系统信息
docker system prune                                       //清理文件，清除所有停止的容器，没有被用的nerwork等
```

## 使用技巧

### 免除 sudo 输入

将当前用户添加到 docker 用户组，再重新打开终端即可

```
sudo groupadd docker                                      //装了docker就默认创建了docker用户组了，可以不用这一步
sudo gpasswd -a longxr docker
systemctl restart docker
newgrp - docker                                           //切换会话或者重启机器
```

### 交互式容器

#### 进入

docker -it ${CONTAINER_NAME}

#### 退出

1. 输入 exit 回车，容器会停止运行
2. ctrl+P+Q (大写模式)，容器不退出

### 进入守护式容器

1. docker attach ${CONTAINER_NAME}，直接进入容器启动命令的终端，不会启动新进程
2. docker exec -it ${CONTAINER_NAME} {command}, 在容器终端打开新终端，并且可以启动新进程

### 主机与容器间的文件拷贝

```
docker cp ${CONTAINER_NAME}:{CONTAINER_PATH} {HOST_PATH}，容器到主机
docker cp  {HOST_PATH} ${CONTAINER_NAME}:{CONTAINER_PATH}，主机到容器
```

## Dockerfile

### 构建自己的 Docker 镜像

```
//通过container来构建image，不推荐
docker container commit
docker image build

//通过Dockerfile生成，Dockfile在当前文件夹下，最后路径就写了个.
docker build -t longxr/centos-vim-new .
```

### Dockerfile 语法

```
FROM scratch                                                //制作base image
FROM centos                                                 //使用 centos base image

LABEL maintainer="test@a.com"                               //添加描述
LABEL version="1.0"
LABEL description="This is description"

WORKDIR /test                                               //设置工作目录，没有则创建，尽量用绝对目录，不要使用RUN cd
ADD hello /                                                 //添加hello文件
ADD test.tar.gz /                                           //添加文件到根目录并解压缩
WORKDIR /root
COPY hello test/                                            //将hello添加到/root/test/hello

RUN yum install wget                                        //添加远程文件使用curl、wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"

ENV MYSQL_VERSION 5.6                                       //设置常量，增加可维护性
RUN apt-get install -y mysql-server="${MYSQL_VERSION}" \
&& rm -rf /var/lib/apt/lists/*

//shell格式
RUN yum update && yum install -y vim                        //执行命令并创建新的Image Layer，所以命令尽量合成一行写
CMD  echo "hello docker"                                    //设置容器启动后默认执行的命令和参数
ENTRYPOINT "hello docker"                                   //设置容器启动时运行的命令，让容器以服务的形式运行

//exec格式
RUN ["apt-get", "install", "-y", "vim"]
CMD ["bin/bash", "-c", "echo hello docker"]                 //如果docker run 指定了其他命令则忽略，多个CMD只有最后的会执行
ENTRYPOINT ["bin/bash", "-c", "echo hello docker"]          //不会被忽略，一定执行

EXPOSE 5000                                                 //暴露5000端口给外部访问
```

## docker-compose

### 常用命令

```
docker-compose up -d                                        //根据docker-compose.yml来创建容器、network、volume并启动，需要看日志的话就不要后台运行 (去掉-d)
                       --scale ${CONTAINER NAME}=3  //同时创建多个容器
docker-compose down                                         //停止并删除up所创建的内容（本地的image不会删除）
docker-compose  start                                       //启动容器
docker-compose stop                                         //停止容器
docker-compose ps                                           //查看当前运行的容器
docker-compose exec mysql bash                              //进入容器中

```

## docker-swarm

### 概念相关

- node: 部署了 docker engine 的 host 实例，既可以是物理主机，也可以是虚拟主机。
- task: 在具体某个 docker container 中执行的具体命令，是 Swarm 中的最小的调度单位。
- service: 是指一组任务的集合，服务定义了任务的属性。
- manager: 负责维护 docker cluster 的 docker engine，通常有多个 manager 在集群中，manager 之间通过 raft 协议进行状态同步，当然 manager 角色 engine 所在 host 也参与负载调度。
- worker: 参与容器集群负载调度，仅用于承载 tasks。

### 集群创建

仅做记录，平时群晖最多到 docker-compose 创建项目就够了

#### 本地虚拟机

```
docker-machine create manger1
docker-machine create worker1
docker-machine create worker2

docker-machine ssh manger1
docker swarm --init --advertise-addr <manger1-ip>         // 如果是安装包装的docker，直接docker swarm --init即可
docker swarm join-token worker                            //加入worker节点需要输入的命令，在init的时候也会打印出来
docker swarm join-token manager                           //加入manager节点需要输入的命令
exit

docker-machine ssh manager2                               //添加manager
docker swarm join --token xxx                             //这里直接粘贴上面的加入manager命令
exit

docker-machine ssh manager3                               //推荐3个或5个manager节点(必须是奇数)来实现高可用。有半数以上manager可用，swarm就可以继续工作
docker swarm join --token xxx                             //这里直接粘贴上面的加入manager命令
exit

docker-machine ssh worker1                                //添加worker
docker swarm join --token xxx                             //这里直接粘贴上面的加入worker命令
exit

docker-machine ssh worker2
docker swarm join-token <token> <ip>:<port>
exit

docker-machine ssh manger1
docker node ls                                            //进入manager节点，可以查看集群node状态
```
