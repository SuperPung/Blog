---
url: netlify-config
title: Hexo&Netlify 博客搭建过程记录
date: 2021-02-21 12:38:35
categories: [技术]
tags: [博客配置, Hexo, Netlify]
---

我是如何从 GitHub Pages 转移到 Netlify 的。

<!--more-->

> 今天在 NexT 官方文档的 [Tag Plugins](https://theme-next.js.org/docs/tag-plugins/) 里面发现了很多高级操作，尝试了一下，新建了「友链」界面。

# 准备工作

{% note info %}

本文的操作是在已经搭建好 GitHub Pages 博客的基础之上的，如果你还没有搭建博客，不妨看一下[这篇文章](https://superpung.com/blog-config-0/)。

{% endnote %}

GitHub Pages 之所以可以发挥作用，是因为在 Hexo 的根目录下的 `_config.yml` 文件中配置了 `deploy` 一项，使得执行 `hexo d` 时将 `public` 内容 push 到 `{username}.github.io` 仓库。

所以网站的生成依靠的是 `public` 文件夹，而部署到 Netlify 则需要将 Netlify 与 GitHub 仓库关联，从仓库中读取内容再进行部署。所以你有两种选择：

1. 直接把 `{username}.github.io` 仓库（也可以改名）和 Netlify 关联，Netlify 直接获取 `public` 内容进行部署（第一种选择）；
2. 将博客源码上传到 GitHub 再和 Netlify 关联，Netlify 通过执行 `hexo g` 生成 `public` 文件夹再进行部署（第二种选择）。

两种方式各有优劣：

- 第一种选择容易操作，且仍可以通过 `{username}.github.io` 访问网站，但无法对博客源码进行版本控制，相比之下迁移仍较麻烦；
- 第二种选择可以对博客源码进行版本控制，且真正实现了随时随地写博客，没有了迁移的困难，但每次都要 `add commit push`，且依赖于 Netlify 服务器的配置。

# Netlify 基本配置

访问 [Netlify 官网](https://www.netlify.com)，点击右上角“注册”：

![netlifyHome](https://i0.hdslb.com/bfs/album/f9d36bcdc2cc50f335a874ad1f4fabf12bea0b01.png)

可以选择用 GitHub 账号注册。

登录后，点击“新建站点”，进入创建站点页面：

![image-20210221162238279](https://i0.hdslb.com/bfs/album/ef2878d3bb5502705ef176508de14bc0fa8b3cad.png)

可以看到，Netlify 告诉你它可以让你「From zero to hero」。

选择 GitHub，关联到 GitHub 仓库。Netlify 的「持续部署」功能，当你 push to Git，Netlify 就会自动运行你的「build tool」并部署。

选择要关联的 GitHub 仓库后，进入下一步：

![image-20210221163103092](https://i0.hdslb.com/bfs/album/d831ac2817a0f6b40f7d6fa56dcd112defa70033.png)

选择要部署的分支后，设置 build 命令和发布目录。

如果你是第一种选择，这两个空留空即可；如果是第二种选择，build 命令应设置为 `hexo g`，发布目录为 `publish/`。

点击部署站点，完成。

# Netlify 高级配置

进入首页，Sites 部分显示了你所有的站点，点击进入后可以对站点和域名进行进一步配置。

## Site settings

- Netlify 部署后会为你的网站自动生成一个随机的域名。在站点设置-“General”中，可以通过修改站点名称来修改域名。

- 在“Build & deploy”中，可以对 build 操作进行设置。
- 在“Domain management“中，可以设置域名。如果你有自己的域名，点击添加后，在域名提供商的域名解析处，通过 CNAME 解析到 Netlify 域名即可。另外，还可以通过修改 Name servers 启用 Netlify DNS。在下方的“HTTPS”处，可以通过 Let’s Encrypt 启用 TLS 证书。

## Deploys

在此处可以看到关于部署的详细记录。
