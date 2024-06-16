---
title: 印象笔记转换 markdown
categories:
  - note
tags:
  - evernote
  - markdown
date: 2024-06-16 23:50:11
---

<Excerpt in index | 摘要>
印象笔记会员快要到期了，看着界面上各种鸡肋功能还限制登录设备，赶紧把数据迁移出来，卸载垃圾软件。 <!-- more -->
<The rest of contents | 余下全文>
以前有时候会有印象笔记网页的剪藏功能，不过实际上收藏的文章很少再去看一遍，只是在囤积电子垃圾罢了，知识还是需要自己整理归纳放到博客或者自己的笔记中才是有用的。从这个角度看，印象笔记完全就是鸡肋，markdown 支持也超弱，后续自己的笔记直接迁移到 Obsidian 。

## 导出笔记

印象笔记默认是不支持直接导出文章为 markdown 的，需要先在印象笔记里选择所有笔记后，菜单选择导出成 enex 格式。

## 笔记格式转换

使用 [evernote2md](https://github.com/wormi4ok/evernote2md) 可以将 enex 格式的文件转成 markdown 格式的文本文件，命令行如下：

```shell
evernote2md (flags) [input] [outputDir]
```

导出后发现有 500+ 各种技术文章，完全想不起来有收集这么多，都没印象了笑死...
