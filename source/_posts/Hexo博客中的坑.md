---
title: Hexo博客搭建中的坑
categories:
  - 问题处理
tags:
  - Hexo
  - blog
date: 2017-04-11 08:35:01
---

<Excerpt in index | 摘要> 
记录坑爹瞬间<!-- more -->
<The rest of contents | 余下全文>
## 问题：Travis-Ci自动构建图标build status unknown
解决：原因是构建的分支不是master分支，默认显示缺省分支master的状态，修改图标链接`svg?branch=master`为`svg?branch=blog`

## 问题：各种在设置_config.yml后出现的问题
解决：检查冒号后面的空格！！！检查冒号后面的空格！！！检查冒号后面的空格！！！

## 问题：自定义域名在blog根目录添加CNAME后，使用hexo d -g不上传CNAME
解决：将需要上传的文件放到blog/source

## 问题：执行 hexo deploy 后,出现 error deployer not found:github
解决：执行npm install hexo-deployer-git —save，再部署

## 问题：YAMLException: bad indentation of a mapping entry
解决：YAML的语法错误，仔细检查冒号后面 要有一个空格

## 问题：部署时候用git连接github提示没有权限被拒绝Permission denied (publickey).
$ ssh -T git@github.com
Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.
解决：重新生成一个SSH key，加到Github上


