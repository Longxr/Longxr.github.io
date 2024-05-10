---
title: git使用ssh多账号配置
categories:
  - 环境搭建
tags:
  - git
  - ssh
date: 2017-05-05 17:36:43
---

<Excerpt in index | 摘要> 
git通过ssh登录时，设置不同的登录账号<!-- more -->
<The rest of contents | 余下全文>

## 生成SSH KEY 
 终端输入：`ssh-keygen -t rsa -C "your_email@example.com" `
 
然后会提示输入公钥的名字，一般就xxx_rsa就行(github_rsa)，生成SSH KEY的时候还要求输入私钥密码`"│Enter passphrase (empty for no passphrase)`，没有特殊需求的话直接回车。密钥的密码什么的完全记不住，设置了以后八成是要重新生成一个密钥了
 
> 有时候会把密钥生成到～／.ssh 的同级目录，最好将密钥移到.ssh文件夹里 

## 将SSH 私钥增加到ssh-agent
`ssh-add ~/.ssh/github_rsa`， 这里会提示输入一次私钥的密码;提示文件不存在的话:

1. 记得去.ssh同级目录找找密钥是不是错放那儿了； 
2. 不识别～，改成全路径：`User/username/.ssh/github_rsa `

查看已经add的SSH KEY： `ssh-add -l；` 

## 将公钥粘贴到自己的github网页设置(服务器) 
1. 登录网页账号，点击头像->Setting->SSH and GPG key，将拷贝的公钥内容粘贴到New key的key里，顺便起个自己能区分的Label 

2. 当有多个账号需要登录服务器或者需要登录多个服务器时，就需要配置SSH的配置文件 

## ssh的配置文件 
ssh client有两个配置文件，/etc/ssh/ssh_config和~/.ssh/config，前者是对所有用户，后者是针对某个用户，两个文件的格式是一样的 

```
Host 名称(自己决定，方便输入记忆的) 
    HostName 主机名 
    User 登录的用户名 
```

假设有两个SSH帐号，一个是github的，一个是bitbucket的，私钥分别是github_rsa、bitbucket_rsa，可以这样写： 

```
#————GitHub————— 
Host longxr-github 
    HostName github.com 
    User longxr 
    IdentifyFile ~/.ssh/github_rsa 

#————Bitbucket———— 
Host longxr-bitbucket 
    HostName bitbucket.org 
    User longxr 
    IdentifyFile ~/.ssh/bitbucket_rsa 
```

> win10直接~/.ssh/github_rsa 就行，~是/c/Users/,没有冒号，也不用反斜杠....win7过来被坑人士

## 登录测试 
```
ssh -T git@github.com 
ssh -T git@bitbucket.org 
```

bitbucket还可以：`ssh -T longxr-bitbucket`，`ssh -T git@longxr-bitbucket`
github不能直接输Host，只能`ssh -T git@longxr-github` 

### 结果 
```
ssh -T git@longxr-bitbucket 
logged in as longxr. 
ssh -T git@longxr-github 
Hi Longxr! You've successfully authenticated, but GitHub does not provide shell access. 
```


