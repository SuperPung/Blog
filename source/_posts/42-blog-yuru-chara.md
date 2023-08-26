---
url: blog-yuru-chara
title: 把萌萌哒看板娘抱回你的博客吧！
date: 2021-02-17 14:17:44
categories: [技术]
tags: [博客配置]
---

可爱的看板娘！

<!--more-->

提到看板娘，不得不介绍一种应用于电子游戏的绘图渲染技术——Live2D。

Live2D 由日本 Cybernoids 公司开发，通过一系列的连续图像和人物建模来生成一种类似二维图像的三维模型，使 2D 的素材实现一定程度的 3D 效果，但只能是一定程度 3D，因为 Live2D 人物无法大幅度转身。

很多知名的动漫都是 Live2D 游戏改编的或者反过来的，例如《我的妹妹哪有这么可爱》、《我的朋友很少》、《樱花庄的宠物女孩》等。

![俺妹](https://i0.hdslb.com/bfs/album/cdc4dbd3881583fb2579ca4d2afdb2378fa6c96b.jpg)

live2d 的[官方网站](https://www.live2d.com/zh-CHS/) 提供了 live2d 开发和编辑软件（如 Live2D Cubism editor 和 Live2D Euclideditor），还有开发使用教程等，对相关制作感兴趣的可以看看。

看板娘是一种职业和习惯称呼，也是 ACG 次文化中的萌属性之一。

看板娘一词源自日语“看板娘（かんばんむすめ）”。其中的“看板”指的是店面招牌，或者是为了宣传、打广告而制作的宣传牌。“看板娘”也就是店面的招牌姑娘，亦即能够提升店面人气和顾客流量的女服务生、女店员等。也就是说，看板娘本身就是一块“活看板”，其本身的魅力就能够起到宣传、拉人气的作用。英语又称之为“Yuru-chara”。

在网页上安装看板娘，大多数都使用的是“hexo-helper-live2d”这一插件。此插件功能较少~~而且没有可爱的 2233 娘~~，所以不用此插件。

网页上安装看板娘，主要分为前端和后端两部分。前端主要是一些 JavaScript 文件，后端则是看板娘的模型及调用接口。

- 前端：https://github.com/SuperPung/super_live2d
- 后端：https://github.com/SuperPung/super_live2d_api

GitHub 不知道从什么时候开始，默认的分支不再是 `master` 而是 `main`，所以 CDN 可能会出现问题，一定要注意。（CDN 的缓存是真的 ex，2.16 改了大半天）

> 参考
>
> - [二次元live2d看板娘中的web前端技术 | 张鑫旭-鑫空间-鑫生活](https://www.zhangxinxu.com/wordpress/2018/05/live2d-web-webgl-js/)
> - [看板娘 - 萌娘百科 万物皆可萌的百科全书](https://mzh.moegirl.org.cn/zh-hans/看板娘)
> - [Live2D看板娘(bilibili-2233) | BlueSky01st's Blog](https://bluesky01st.js.org/posts/2afd864b.html)
> - [在网页中添加 Live2D 看板娘 | 米米的博客](https://zhangshuqiao.org/2018-07/在网页中添加Live2D看板娘/)
