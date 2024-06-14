---
title: Hexo 3.5.0 迁移到 Github Action 踩坑记录
categories:
  - Hexo
tags:
  - Hexo
  - workflows
date: 2023-05-22 22:34:17
---

<Excerpt in index | 摘要>
忘记写博客两年半，域名续费才想起来博客的存在哈哈哈哈，顺便吐槽下 Hexo 官方文档都不维护更新下内容坑啊 <!-- more -->
<The rest of contents | 余下全文>

在我昨天玩游戏心血来潮又想写点东西的时候，Clone 下来久违的博客仓库，提交完文章后，发现博客完全不更新的，这个时候就发现事情并不简单 Σ(っ °Д °;)っ

![HexoGithubAction-2023-05-22-23-41-06](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-22-23-41-06.png)

登录 travis-ci 一看，最近一次成功的自动构建记录都是 2 年 前的事情了，上方一行红色文字提示我信用额度为负，好家伙。。。许久不见，travis-ci 的构建已经变成收费功能了。莫得法子，只能换一家白嫖了，正好 Github 也有 Github Action 是可以用于自动构建的，于是挥泪告别 travis-ci (哭了，我装的(￣ ▽ ￣)")

# workflows

网上看了一圈教程，workflows 写法五花八门，有用 SSH key 的，有用 GitHub Token 的，最后还是决定按照官方文档来了，[相关链接戳这里](https://hexo.io/zh-cn/docs/github-pages)

![HexoGithubAction-2023-05-22-23-50-33](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-22-23-50-33.png)

# 踩坑处理

提交 `.github/workflows/pages.yml` 后，自动触发构建，发现 Action 报错:

## Invalid workflow file: .github/workflows/pages.yml#L2

![HexoGithubAction-2023-05-22-23-59-02](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-22-23-59-02.png)

定睛一看，啊这...原来第一行是注释么，怎么不加 **#号**的，复制粘贴害人呀，改完又报错:

## Node.js 12 actions are deprecated. Please update the following actions to use Node.js 16: actions/checkout@v2, actions/setup-node@v2, actions/cache@v2

![HexoGithubAction-2023-05-23-00-04-41](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-04-41.png)
搜索了下报错，找到相关 [issue](https://github.com/actions/checkout/issues/1047), 看着意思是需要把文档中的 `actions/checkout@v2` 改成 `actions/checkout@v3`，`actions/setup-node@v2`、`actions/cache@v2` 也是同理。不过感觉以后可能还有升级...可以跳到仓库的主页看看目前最新的是哪个版本，比如 [actions/checkout](https://github.com/actions/checkout):

![HexoGithubAction-2023-05-23-00-07-55](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-07-55.png)

改好提交，改完继续报错...

## npm ERR! Missing script: "build"

![HexoGithubAction-2023-05-23-00-10-42](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-10-42.png)

我寻思我也没改过 npm 的配置，咋就报错了。于是又在本地初始化一个新的 Hexo 仓库对比下，看下有何不同之处，对比发现 `package.json` 确实不一样，新版除了依赖模块还加了 `scripts` 部分

![HexoGithubAction-2023-05-23-00-25-55](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-25-55.png)

加上 `scripts` 部分后构建不会报错了，但是构建完成 push 到仓库后，网站全白啥也没有，查看仓库文件 html 全是 0 kb，我文章跑哪去了 ？？

![HexoGithubAction-2023-05-23-00-31-17](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-31-17.png)

## 生成 html 为空

搜索一番发现，其他人也碰到了这个问题，是由于 **node 版本太高了，hexo 还不支持**
![HexoGithubAction-2023-05-23-00-34-45](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/HexoGithubAction-2023-05-23-00-34-45.png)

将 `.github/workflows/pages.yml` 中的 node 版本改回 12 后，再次构建，我的博客终于又回来了

# .github/workflows/pages.yml

附上修改后的配置文件，方便其他有需要升级老版本 Hexo 的朋友

```yml
name: Pages

on:
  push:
    branches:
      - blog # 源码所在分支

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3 # 有报错的话可能需要升级到最新的脚本
      - name: Use Node.js 12.x
        uses: actions/setup-node@v3
        with:
          node-version: '12' # 改为 Hexo 所需的 node 版本
      - name: Cache NPM dependencies
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build # 检查 package.json 中是否有相关的 scripts
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

# 参考链接

- [在 GitHub Pages 上部署 Hexo](https://hexo.io/zh-cn/docs/github-pages)
- [hexo 生成的 html 文件为空的问题](https://alanlee.fun/2021/02/28/hexo-empty-html/)
