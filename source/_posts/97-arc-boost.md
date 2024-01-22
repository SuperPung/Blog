---
url: arc-boost
title: 使用 Arc 浏览器自定义网页
date: 2023-01-27 13:52:06
categories: [技术]
tags: [Arc 浏览器]
---

方便、快捷地自定义网页，无需安装任何插件。

<!-- more -->

## 什么是 Arc 浏览器

在上一篇文章 —— [2022 年度总结](/rev-2022) 中，我提到了去年比较满意的一款产品：Arc 浏览器。它是一款基于 Chromium 的浏览器，有很多传统浏览器没有的新功能。这里不过多介绍它的其他功能，主要讲一讲自定义网页 —— 即 [Arc Boost](https://youtu.be/53KQ2wUZG2s)。

## 为什么要自定义网页

我喜欢使用浏览器，只要浏览器能做的，我一般不会下载 app。很多网站的布局、设计和推荐的内容都不是我想要的，我希望能够自定义网页，只看到我想要的内容。比如在用 Bilibili 时，视频上方的推荐搜索、直播间花里胡哨的活动通知和礼物等，我都不想看到。这些内容在 app 中是不可以自定义的，但正因为是浏览器，才有了自定义网页的可能。

## 为什么不...？

### 为什么不直接修改网页源代码？

这样做的话，每次网页刷新都需要重新修改，而且不同路由下的网页都需要修改，比较麻烦。

### 为什么不直接安装插件？

{% note info %}

上文和下文提到的“插件”指的就是浏览器的扩展（extension）。

{% endnote %}

当然，对于 Bilibili 网站，有很多出色、强大的自定义插件可供使用。但插件提供了太多我不需要的功能，提供的需要的功能实现又过于复杂也不是我想要的样子，而且向插件中添加我想要的功能又太麻烦，最后成了为了自定义网页而去自定义插件。

## 如何快速利用 Arc Boost 自定义网页（以 Bilibili 为例）

首先，你要有一台运行 macOS 的电脑，因为 Arc 浏览器暂时只支持 macOS。

其次，你需要安装 Arc 浏览器，这里是邀请码：

> hey, here’s an invite to Arc, the browser I was telling you about!
>
> https://arc.net/gift/44d83098

在浏览器中打开 [Bilibili](https://bilibili.com)，在侧边栏的右下角 `+` 号中选择 `New Boost`，即可进入 Arc Boost 编辑页面。

![Arc Boost 编辑页面](https://i0.hdslb.com/bfs/album/1dbcab3fdcf8e8d848f326145da195165d14c18e.png)

点击 `Style` - `A specific website`，在窗口中将 `www.bilibili.com` 改为 `bilibili.com`，点击 `Create Boost`，即可进入 `styles.css` 编辑页面。

{% note info %}

因为 Bilibili 空间、直播间等页面的三级域名都不是 www，如果这里只写 www，会导致自定义 css 在这些页面无法生效。

{% endnote %}

![Arc Boost css 页面](https://i0.hdslb.com/bfs/album/22e9ac3d7f36682460d2259855ce4983f5d0645f.png)

这里就是自定义网页的地方了，你可以在这里添加你想要针对 Bilibili 的 css 代码，浏览器会自动覆盖掉原网页的 css。

{% note info %}

你应该对 css 有一些比较基础的了解。如果完全不了解，Arc 浏览器还贴心地提供了很多教程，在右上角的 Handbook 中可以找到。

{% endnote %}

下面以隐藏 Bilibili 网页上方的推荐搜索为例，这里是修改前的网页：

![修改前](https://i0.hdslb.com/bfs/album/3bf9db6c793b12ddbfb00f839b51911e940be08f.png)

用开发者工具找到搜索框的元素路径，这里是 `#nav-searchform > div.nav-search-content > input`。

删除原有的 css 代码，添加如下代码：

```css
#nav-searchform > div.nav-search-content > input::placeholder {
  color: transparent;
}
```

浏览器会自动保存并自动刷新，修改后的网页如下：

![修改后](https://i0.hdslb.com/bfs/album/7d216ad778c1d1415dd9115f37a6f70260f590af.png)

而且此后只要通过 Arc 浏览器访问 Bilibili，都会自动隐藏搜索框的推荐搜索。

是不是很简单！

## 修改更多网页

这三行代码可以隐藏 Bilibili **首页** 上方的推荐搜索，但在其他页面可能不生效。因为 Bilibili 其他页面的结构可能有所不同，所以需要到不生效的页面再次找到对应的元素路径，然后添加 css 代码。

这里以直播间页面 https://live.bilibili.com/ 的搜索框路径为例：

```css
#nav-searchform > div.p-relative.search-bar.over-hidden.border-box.t-nowrap > input::placeholder {
  color: transparent;
}
```

应该可以隐藏全部搜索框的推荐搜索了。

说到直播间，默认的 Bilibili 直播间是这样的：

![Bilibili 默认直播间](https://i0.hdslb.com/bfs/album/d675a80cd2b2978ee3b45771e9460a0b861306c6.png)

比如我只想看 v，不喜欢充值、送礼物、打赏、买周边、上舰，那么上面的礼物星球、限时领取、购物提示，以及下面的小橙车、一排礼物、上舰提示、充值按钮，都是我不想看到的。这时就可以找到它们的元素路径，然后添加 css 代码隐藏它们。

```css
#head-info-vm > div > div > div.lower-row > div.left-ctnr > div.gift-planet-entry {
  display: none;
}
#head-info-vm > div > div > div.lower-row > div.left-ctnr > div.activity-gather-entry.activity-entry.s-activity-entry {
  display: none;
}
#gift-control-vm > div > div.vertical-middle.dp-table.section.right-part.p-relative {
  display: none;
}
#gift-control-vm > div > div.left-part-ctnr > div.ecommerce-entry.gift-left-part {
  display: none;
}
#sections-vm > div.section-block.f-clear.z-section-blocks > div.left-container > div.flip-view.p-relative.over-hidden.w-100 {
  display: none;
}
#shop-popover-vm, #my-dear-haruna-vm {
  display: none;
}
```

修改后：

![Bilibili 修改后直播间](https://i0.hdslb.com/bfs/album/9434bae5a17737cf2d8529f4c387e2d5e2c72be4.png)

是不是简洁很多！这下可以专心看直播了。

同理，你可以修改任何网页，只要找到对应的元素路径，然后添加 css 代码就可以了。

## 这是怎么实现的？

实际上，Arc Boost 就是为浏览器添加了一个插件。比如刚才我们添加的对于 bilibili.com 的 Boost，可以在已安装的插件中找到：

![已安装的插件](https://i0.hdslb.com/bfs/album/0c2f3aff2006a0d43b6543cde2874fb03e3c18a8.png)

在插件管理中，可以看到具体信息，包括插件的大小、权限和存储位置等：

![Arc Boost 详情](https://i0.hdslb.com/bfs/album/3f15f51105a1fc8846e54e7ee0520dc61227ae73.png)

上面显示 < 1 MB，其实也就 2 KB。

![Arc Boost 文件夹](https://i0.hdslb.com/bfs/album/d366149c8c08efe6aa89c3e0a519c08d043e96d8.png)

## 如何发挥 Arc Boost 更大的作用？

得益于 Arc 浏览器优秀的套壳技术，对于已经添加的 Boost，可以在作用的网页右上角的插件列表（Extensions）中看到，且单独以“Boosts”列出，可以一键删除、修改或置顶，不需要单独在 Finder 中打开并修改。

Arc Boost 的作用远不止于此。本文只介绍了它的最简单的用法 —— 改变网站样式，实际上，正如一开始进入 Boost 所见，它还可以通过编写 JavaScript 代码替换、注入网站内容，或者做任何事情。它的存在将浏览器插件的开发门槛降到了极低，让任何人都可以轻松地编写并使用适合自己的浏览器插件，不依赖其他工具。

https://arcboosts.com/ 收录了很多有趣的 Boost，你可以在这里探索适合自己的，或者自己编写一个并上传上去。如果你有更多关于 Arc Boost 或 Arc 浏览器 的想法，欢迎在下方留言交流。

## 题外话

{% note info %}

如果你只想看关于本文主题的内容，可以跳过这一部分。

{% endnote %}

细心的小伙伴应该已经发现了，我的博客域名又又又换了。从最开始的 .xyz 到后来的 .cn，再到现在的 .com，以后应该不会再换了。这次换域名的原因是，之前在 [Value Domain](https://www.value-domain.com/) 上面一元买到了 6 个域名，现在的这个域名就是其中之一。.com 域名确实更加通用，而且不用放在国内的阿里云或腾讯云上（相比 .cn），我转移到了 Cloudflare 上，体验也更好。原来的 .cn 域名做了跳转，所以这次迁移暂时也不会影响到原来的链接（为什么说是“暂时”呢，因为 .cn 域名过期之后大概率不会再续费了，所以到时候原来的链接肯定会失效）。唯一受影响的就是之前不蒜子统计的浏览数据了，不过也没什么关系，新的开始，新的计数。
