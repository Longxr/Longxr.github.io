---
title: Python Anaconda 使用
categories:
  - Python
tags:
  - Python
  - Anaconda
  - conda
date: 2024-05-25 21:25:34
---

<Excerpt in index | 摘要>
Anaconda 是一个开源的 Python 发行版本，用它来管理 python 环境。 <!-- more -->
<The rest of contents | 余下全文>
不同项目用到的 Python 版本不同，依赖的三方库也可能不同，如果都在默认的 Python 环境安装依赖可能会有冲突，这时候就可以用 Anaconda 针对不同的项目设置不同的 Python 版本以及下载不同版本的依赖库。

## 安装

官网下载安装包即可，[miniconda 下载](https://docs.anaconda.com/free/miniconda/)

### 添加镜像源

如果能翻墙可以直接忽略，镜像源主要处理下载被墙的问题。

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

//更新conda，保持conda最新
conda update conda

//更新anaconda
conda update anaconda
```

## 虚拟环境管理

不同的 Python 环境是独立的，可以分别装不同的 Python。

```shell
//创建虚拟环境，后面加上 anaconda 参数, 会把base的基础包一起安装
conda create -n env_name python=x.x

//列出当前所有环境，列出的环境中，当前环境前面有个*号
conda info --envs

//激活环境
conda activate env_name

//取消激活，取消激活当前环境的话，后面的环境名也可以省了
conda deactivate env_name

//删除环境
conda remove -n env_name --all

//克隆环境
conda create -n new_env_name --clone env_name

//导出环境配置，导出的配置文件会在当前目录下
conda env export > environment.yml

//通过配置文件创建环境
conda env create -f environment.yml
```

## 包管理

### 使用 conda 管理当前环境包

包的管理是针对当前环境的，所以最好是用 Anaconda Prompt 来操作，在终端位置前可以看到当前虚拟环境名，在其他终端也可以通过`conda info --envs`来查看当前环境

```shell
//在当前环境安装包
conda install <package_name>

//删除当前环境的包
conda remove <package_name>

//更新当前环境的包
conda update <package_name>

//更新当前环境所有包
conda update --all

//查看当前环境安装的包
conda list

//查找包，模糊匹配
conda search <package_name>

//查找包，精确查找
conda search --full-name <package_name>

```

### 使用 pip 管理当前环境包

用 pip 和 conda 安装的包，都可以通过 list 命令查看到已安装包时看到，混着用貌似也没事，看了下安装路径都是`C:\Users\Administrator\Anaconda3\envs\<env_name>\Lib\site-packages`

```shell
//在当前环境安装包
pip install <package_name>

//删除当前环境的包
pip uninstall <package_name>

//更新当前环境的包
pip install <package_name> --upgrade

//更新当前环境所有包（仅供参考，我是用的conda更新所有包）
pip freeze --local | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip install -U --user

//查看当前环境安装的包
pip list

```

### 使用 conda 管理指定环境包

conda 对包的管理在不切换环境时也可以进行，只需要在对应命令后面加上对应的虚拟环境名即可

```shell
//在指定环境安装包
conda install -n <env_name> <package_name>

//删除指定环境的包
conda remove -n <env_name> <package_name>

//更新指定环境的包
conda update -n <env_name> <package_name>

//更新指定环境所有包
conda update -n <env_name> --all

//查看指定环境安装的包
conda list -n <env_name>

```
