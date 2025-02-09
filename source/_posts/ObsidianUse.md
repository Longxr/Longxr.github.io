---
title: Obsidian 使用
categories:
  - Obsidian
tags:
  - Obsidian
date: 2025-02-09 22:45:21
---

<Excerpt in index | 摘要>
逐步迁移之前的笔记到 Obsidian，记录下常用的功能。 <!-- more -->
<The rest of contents | 余下全文>

## 知识管理流程

- 学习知识
- 保存知识
  - 输出，Markdown
  - 整理，双链
- 使用知识
- 共享知识
- 创新知识

## Markdown 语法

- ### 标题，最多支持 6 个#号
- **加粗**
- _斜体_
- ~~删除线~~

---

- 列表，创建好项后可通过 Tab 变成子项
  - 有序列表
    1. 序号 1
    2. 序号 2
  - 无序列表
    - 序号 1
    - 序号 2
- > 引用内容
- [链接](https://www.bilibili.com/video/BV1V34y1k7St)
- [x] 待办事项，Ctrl+L

## 快捷键

命令面板，Ctrl+P
编辑模式/预览模式切换，Ctrl+E

## 双链

通过两个方括号可以引用文章，#号可以定位标题，|添加别名，最前面加!可以直接在当前文章显示链接的内容

![[DockerCommand#常用命令|Docker 常用命令]]

## 搜索

### 快捷键

- 搜索当前文档： Ctrl + F
- 搜索整个资料库：Ctrl + Shift + F

### 搜索技巧

- 直接搜索关键词
- 多个关键词搜索，通过空格间隔（AND）, 只要一个关键词包含用 OR
- 指定搜索范围：
  - 搜索文件名： file:word
  - 搜索文本内容： content:word
  - 搜索标签：tag:#word
  - 搜索同一行中的多个关键词：line:word1 word2
  - 搜索同一章节中的多个关键词：section:word1 word2
  - 搜索同一段落中的多个关键词：block:word1 word2
- 搜索任务：
  - 搜索任务：task:"word"
  - 搜索未完成任务：task-todo:"word"
  - 搜索已完成任务：task-done:"word"

### query

```query
搜索技巧 搜索任务
```

## Dataview 查询

Dataview，Obsidian 资料库的查询工具/插件

- 查询对象： Obsidian 数据库
- 查询依据： YAML 数据 / 文档 Metainfo

- YAML
  - 位于 Markdown 文件开头， 首尾用 --- 包含的部分
  - 默认支持的字段
    - tags
    - publish
    - cssclass
    - aliases
  - 自定义字段
    - category
    - date
    - time
    - rating
- Obsidian 文件属性
  - file.name: 文件标题（字符串）
  - file.floder: 文件所属文件夹路径
  - file.path: 文件路径
  - file.size: 文件字节大小
  - file.ctime: 文件创建时间
  - file.mtime: 文件修改时间
  - file.cday: 文件创建日期
  - file.mday: 文件修改日期
  - file.tags: 笔记中所有标签数组
  - file.etag:s 除去子标签的数组
  - file.inlinks: 指向此文件的所有传入链接的数组
  - file.outlinks: 此文件所有出站链接的数组
  - file.aliase:s 文件别名数组
  - file.day: 如果文件名中有日期，那么就会以这个字段显示
- 任务属性
  - Task 会继承所在文件的所有字段，比如 Task 所在页面中已经包含了 rating 信息了，那么 Task 也有
  - completed: 任务是否完成
  - fullyCompleted: 任务以及子任务是否完成
  - tex:t 任务名
  - line: 任务所在行
  - path: 任务所在路径
  - sestion: 连接到任务所在区块
  - link: 连接到任务最近的可连接的区块
  - subtasks: 子任务
  - real: 如果为 true, 则是一个真正的任务，否则就是一个任务之前或之后的元素列表
  - completion: 任务完成的日期
  - due: 任务到期时间
  - created: 创建日期
  - annotated: 如果任务有自定义标记则为 ture， 否则为 false

### 语法

以 三个点开头，后面跟上 dataview 表明这是 dataview 代码

```代码类型
[list|table|task] field1, field2 as myfield,
FROM #tag or "floder" or [[link]] or outgoing([[link]])
WHERE field [> | < | =] [field2 | literal value] (and field1 ...) (or field3...)
SORT field [ascending | descending | asc | desc] (ascending is implied if provided)
```

### 示例

#### 文件夹查询

```dataview
list
FROM "02 学习"
```

#### 标签查询

```dataview
TABLE file.size as 文件大小, file.ctime as 创建时间
FROM #工具
SORT file.name desc
```

## 白板（画布）

任意内容都可以放入白板中作为卡片展示，比如笔记、图片、网页、文本等。

### 新建

目录右键新建画布，或者 Ctrl + P，输入白板

## Callout

> [!NOTE] 提示内容
> 就是 Markdown 的引用，展示上有优化，展示类型可以右键后选择 Callout type 修改
