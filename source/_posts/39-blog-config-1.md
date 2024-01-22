---
url: blog-config-1
title: 博客配置问题
date: 2021-02-14 11:09:58
categories: [技术]
tags: [Hexo, NexT, GitHub, blog]
---

GitHub&Hexo&NexT Blog 配置过程记录（下）：其他配置及问题

<!--more-->

> 在 [上一篇文章 ](/blog-config-0)中，已经搭建好了博客网站，这篇文章就来记录一下玩博客一年多来遇到的问题吧。
>
> 下文添加的功能，建议添加完成后通过 `hexo g && hexo s` 本地预览。
>
> P.S. 今天看维基百科的时候，发现除了简体中文和繁体中文外，竟然还有文言文版本的“[维基大典](https://zh-classical.wikipedia.org/wiki/維基大典:卷首)”：
>
> > **歡迎**
> >
> > 自永樂修典，四庫編成，古今圖冊，收攬完備。惟近世曉覺道理，百家爭鳴。西學東漸，各有始末。士紳茫茫，遠不及逮。疑古者眾，怨舊者多。於是斥逐儒術，貶抑性理。殷周之明，莫非妖言；泰西末流，敬為上賓。崇外若此，至今百年。會西國志士，立典於網絡，開共筆之先河。吾人乃竊取一處，成以文言，謀復古法，載新世之大道，以揚中華文理，興千年舊邦，故亟需善古文而博今道者。願足下能同遊，共為大典，修先世之廢道，著當今之新知。
> >
> > 又，古文維基大典，以其從古，多有異於外文，宜先閱[典章](https://zh-classical.wikipedia.org/wiki/維基大典:五柱)、[凡例](https://zh-classical.wikipedia.org/wiki/幫助:凡例)、[章法](https://zh-classical.wikipedia.org/wiki/維基大典:章法)，以求壹道。
> >
> > 如有垂詢賜教，請至[會館](https://zh-classical.wikipedia.org/wiki/維基大典:會館)。[燕語](https://zh-classical.wikipedia.org/wiki/維基大典:燕語閣)如故，亦無不可。
> >
> > -- 孔明居士 二〇二一年二月一四日 （日） 〇八時五八分 (UTC)
>
> 真的是惊喜。
>
> P.S. 21 年 2 月 17 日更新：为了方便阅读，将内容拆分。
>
> P.S. 21 年 2 月 20 日更新：重新整理了部分内容。

# 迁移博客遇到的问题

上一篇文章提到过，NexT 一共有三个不同的仓库：

| 版本            | 年份        | 仓库                                          |
| --------------- | ----------- | --------------------------------------------- |
| v5.1.4 或更低   | 2014 ~ 2017 | https://github.com/iissnan/hexo-theme-next    |
| v6.0.0 ~ v7.8.0 | 2018 ~ 2019 | https://github.com/theme-next/hexo-theme-next |
| v8.0.0 或更高   | 2020 ~ ?    | https://github.com/next-theme/hexo-theme-next |

遗憾的是，每个新仓库的创建者都没有 Archive 旧仓库的权限。因此许多网络上的教程并不能区分这三个仓库的区别，特别是后两个名称相近的。为了避免安装错误的 NexT，请务必严格按照最新仓库 README 中提供的安装方式进行操作。

我刚开始搭建博客的时候，是 2020 年 1 月，当时没有仔细了解，安装的是 NexT v5 版本，也就是第一个仓库的版本。两个多月之后，由于电脑的问题，开始了第一次迁移。多亏了 GitHub，我的文章不至于全部消失。其实迁移博客只需要把原博客根目录下的 source 文件夹拷贝到新建好的博客根目录，再配置一下主题即可。当时由于 v5 功能过少~~（其实我只是为了网站页脚的小图标能动起来）~~，迁移时将 NexT 升级到了 v7。迁移后有了动态的图标，还有 Safari 的彩虹效果等新功能。

2021 年 2 月，迁移博客时升级到了 NexT v8。跨版本的升级可能并不顺滑（例如由 v5.1.4 或 v7.8.0 升级至 v8.0.0），请备份配置文件及修改过的文件（例如自定义模板文件）后，重新安装新的主题。具体操作请阅读文档： https://theme-next.js.org/docs/getting-started/upgrade.html

这次升级（从 v7 到 v8）有很多重大改变：

- **v7.4.2 Nunjucks 引擎**

  鉴于 swig 缺乏维护，NexT 自 7.4.2 版本开始，使用 Nunjucks 代替 swig 作为模版引擎。如果此前根据 swig 的语法写过自定义内容，请在更新前确认它们是与 Nunjucks 兼容的，否则会报错，且生成的页面为空白。例如， Nunjucks 只支持 `and` 运算符，需要替换掉 swig 中的 `&&`。见 http://mozilla.github.io/nunjucks/getting-started.html

  Hexo 5.0 版本移除了对于 swig 模版的支持，改为独立的 hexo-renderer-swig 插件。如果你在使用旧版本的 NexT，并发现 Hexo 生成的 html 中输出了模版源码，请根据 NexT 官网「Hexo 与 NexT 兼容性」部分的内容选择升级 NexT 或降级 Hexo。我们不建议继续使用旧版本的 NexT。

- **v7.6.0 `auto_excerpt`**

  自 7.6.0 版本开始，`auto_excerpt` 功能被移除，因为它并不属于 Hexo 主题应当负责的内容，并对 NexT 的开发造成了麻烦。我们推荐通过 `<!-- more -->` 来精确控制 Read More 的位置；或者设置 `excerpt_description` 然后为每篇文章指定 `description`。当然，也可以自行安装第三方插件：

  https://github.com/chekun/hexo-excerpt
  https://github.com/ashisherc/hexo-auto-excerpt

  作出以上改动后，请执行 `hexo clean`。

- **v8.0.0-rc.1 自定义图标**

  NexT 主题自 8.0.0 版本开始，将自带的 Font Awesome 图标库由 4.7.0 版本升级为了 5.13.0 版本。此次升级并不向下兼容，请修改配置文件中与 Font Awesome 相关的内容，否则图标可能无法正常显示。

  全部可选图标在此： https://fontawesome.com/icons

- **v8.1.0 移除 Valine**

  Valine 评论系统出现了一些令人担忧的问题：

  - NexT 团队曾多次收到关于 Valine 评论系统存在隐私泄露问题的反馈；
  - Valine 自 1.4 版本起不再开源，因此 NexT 团队无法对 Valine 评论系统 Debug。并且发布的打包版本中存在未告知用户的百度统计代码；
  - 2020 年 11 月下旬出现了针对 Valine 评论系统的攻击。

  考虑到这些问题已经严重影响到 NexT 用户的数据安全，我们决定将其移除，需要继续使用的用户请安装插件： https://github.com/next-theme/hexo-next-valine（插件的配置项使用驼峰命名，与 Valine 本身一致，需要注意将 `appid` 和 `appkey` 改为 `appId` 和 `appKey`）

  鉴于以上原因，如果在使用 Valine 时出现*任何*问题，请在这里反馈： https://github.com/xCss/Valine/issues

  迁移到 Disqus： https://github.com/YunYouJun/valine-to-disqus

- 欢迎加入 Telegram 群，讨论问题更方便

  中文群：https://t.me/theme_next_cn

# 添加点击特效

具体操作参见[这篇文章](/blog-click-effects)。

# 添加 Safari 彩虹效果

从 OS X Yosemite 系统开始，Safari 浏览器的顶部工具栏加入了类似 iOS 中的半透明毛玻璃效果。Safari 会自动根据页面的颜色来显示工具栏的毛玻璃特效颜色。

在 NexT 的某一版本中内置了这一功能（v7.7.2 中仍保留）：

```yaml
# Hide sticky headers and color the menu bar on Safari (iOS / macOS).
safari_rainbow: true
```

遗憾的是，最新版 NexT 移除了这一功能。还可以通过自定义配置实现。

在 `_config.next.yml` 文件中，开启 `custom_file_path` 中的 `style`，并在 `/source/_data/` 目录下新建 `styles.styl` 文件：

```stylus
@media screen and (-webkit-min-device-pixel-ratio: 0) {
    body:before {
        right: 0;
        top: 0;
        left: 0;
        height: 100px;
        z-index: 2147483647;
        position: fixed;
        content: "";
        display: block;
        -webkit-transform: translateY(-99.99px);
        background: linear-gradient(124deg,
        #FF0000,
        #FF7F00,
        #FFFF00,
        #7FFF00,
        #00FF00,
        #00FF7F,
        #00FFFF,
        #007FFF,
        #0000FF,
        #7F00FF,
        #FF00FF,
        #FF007F,
        #FF0000);
        animation: rainbow 15s ease infinite;
        background-size: 1000% 1000%;
    }
}
@keyframes rainbow {
    0% {
        background-position: 0% 80%;
    }
    50% {
        background-position: 100% 20%;
    }
    100% {
        background-position: 0% 80%;
    }
}
```

完成。

> 参考
>
> - [Safari顶栏彩虹效果 | 米米的博客](https://zhangshuqiao.org/2018-11/Safari顶栏彩虹效果/)

# 添加今日诗词

喜欢诗词的同学绝对要添加的一个功能。

其实很简单，只是调用一个现成的接口，并不会自己写一个诗词库。

在 `_config.next.yml` 文件中，开启 `custom_file_path` 中的 `footer`，并在 `/source/_data/` 目录下新建 `footer.njk` 文件（可以是其他你想放置的位置）：

```html
<span id="jinrishici-sentence">正在加载今日诗词....</span>
<script src="https://sdk.jinrishici.com/v2/browser/jinrishici.js" charset="utf-8"></script>
```

完成。

> 参考
>
> - [通用简单安装代码 | 今日诗词开放接口 - 今日诗词](https://www.jinrishici.com/doc/#json-fast-easy)

# 添加文章置顶

具体操作参见[这篇文章](/blog-top-article)。

# 添加文章加密

如果你想把文章只对特定人可见，那么可以试试对文章加密。

安装插件：

```zsh
npm install --save hexo-blog-encrypt
```

使用时只需在文章头添加 `password` 字段：

```markdown
---
title: test
date: 2021-02-14 11:09:58
password: hello
---
```

完成。

> 参考
>
> - [hexo-blog-encrypt](https://github.com/D0n9X1n/hexo-blog-encrypt)

# 添加自定义页面

如果你想在博客子目录添加一个自定义的 html 页面，不妨继续看。

受菜单栏启发，可以将自定义页面放到菜单栏。在 `_config.next.yml` 文件的 `menu` 中，按照 `Key: /link/ || icon` 格式添加一项。

比如添加 `love: /love/ || fa fa-heart`，即在菜单栏添加了一个名为“love”、路径为 `/source/love/`、图标为 `fa fa-heart` 的菜单。

接下来在对应路径处放置写好的 html 文件 `index.html`。此时生成预览后，可以发现自定义的页面上方仍有菜单栏、下方有网站页脚。这是 Hexo 将自定义页面也进行了渲染的结果。

如果不想对自定义页面渲染，可以按照上一篇文章提到的，在 `_config.yml` 文件的 `skip_render` 中添加 `love/index.html`。或者在 `index.html` 开头添加：

```html
---
layout: false
---
```

完成。

# 添加评论功能

如果你想更好地和读者互动，评论功能必不可少。当然，写出足够优秀的文章是更好互动的前提。

评论系统用得最多的就是 Valine 和 Disqus。Valine 由于各种问题而被最新版 NexT 移除，但考虑到界面的简洁程度、网络环境影响的访问速度，我还是决定使用 Valine。

Valine 是一个快速简洁又高效的、依赖于 LeanCloud 的、无后端的评论系统。通过 `npm` 安装插件：

```zsh
npm install next-theme/hexo-next-valine
```

注册 [LeanCloud](https://www.leancloud.cn)，创建应用，找到“应用 Keys”中的 `AppID` 和 `AppKey` 并记住。

在 `_config.next.yml` 中添加：

```yaml
# Valine
# For more information: https://valine.js.org, https://github.com/xCss/Valine
valine:
  enable: false
  appId:  # your leancloud application appid
  appKey:  # your leancloud application appkey
  serverURLs: # When the custom domain name is enabled, fill it in here
  placeholder: Just go go # comment box placeholder
  avatar: mm # gravatar style
  meta: [nick, mail, link] # Custom comment header
  pageSize: 10 # pagination size
  visitor: false # leancloud-counter-security is not supported for now. When visitor is set to be true, appid and appkey are recommended to be the same as leancloud_visitors' for counter compatibility. Article reading statistic https://valine.js.org/visitor.html
  comment_count: true # If false, comment count will only be displayed in post page, not in home page
  recordIP: false # Whether to record the commenter IP
```

完成。

可以通过 Lean Cloud 管理评论。

> 参考
>
> - [hexo-next-valine](https://github.com/next-theme/hexo-next-valine)

# 添加看板娘

具体操作参见[这篇文章](/blog-yuru-chara)。

# 写文章

具体内容参见[这篇文章](/markdown)。

# 插入音乐及视频

如果你想在文章中插入喜欢的音乐或视频，可以在音乐或视频网站找到嵌入代码，Markdown 支持 html。

## [网易云音乐](https://music.163.com/)

点击“生成外链播放器”，选择“iframe 插件”，选择尺寸和播放模式后，复制 HTML 代码：

```html
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1457707546&auto=1&height=66"></iframe>
```

效果如下：

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1457707546&auto=1&height=66"></iframe>

[QQ 音乐](https://y.qq.com)不行。

## [bilibili](https://www.bilibili.com/)

点击“分享”，复制“嵌入代码”：

```html
<iframe src="//player.bilibili.com/player.html?aid=69047664&bvid=BV1nJ411T7m6&cid=119667952&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```

效果如下：

<iframe src="//player.bilibili.com/player.html?aid=69047664&bvid=BV1nJ411T7m6&cid=119667952&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

[YouTube](https://www.youtube.com)、[优酷](https://www.youku.com)、[爱奇艺](https://www.iqiyi.com)、[腾讯视频](https://v.qq.com) 等类似。

# 插入图片遇到的问题

具体操作参见[这篇文章](/blog-insert-images)。

# 写在最后

本来只是想简单记录一下玩博客的想法，没想到正逢辛丑牛年春节，已经陆陆续续写了四天。如果你看到了这里，那一定是缘分。如果你感觉我的经历对你有所帮助的话，请留言或者赞赏或者联系我，让我知道帮助到了你。这是对我的莫大鼓励。

当你开始尝试自己搭建博客的时候，你会发现，世界上有千千万万人和你一样，在 GitHub、Hexo 和 NexT 之上，用自己的智慧和个性让互联网更加繁荣多彩。
