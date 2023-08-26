---
url: blog-config-0
title: GitHub Pages&Hexo&NexT 博客搭建过程记录
date: 2021-02-11 09:17:47
categories: [技术]
tags: [博客配置, GitHub Pages, Hexo, NexT]
---

你也想拥有这样的博客吗？（上）基本配置

<!--more-->

> Hi，你目前进入的[这个博客网站](https://superpung.com/)，就是按照这篇文章来配置的。
>
> 今天是 2021 年 2 月 11 日，除夕，提前祝看到这里的你新年快乐。*（虽然没人看到）*
>
> <iframe src="//player.bilibili.com/player.html?aid=971338954&bvid=BV1zp4y1W7qf&cid=294929309&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
>
> [这个博客网站](https://superpung.com/)最初建成于 2020 年 1 月 14 日左右，陆续由于换了四次电脑，博客也迁移了三次。从建成至今，不知不觉已经过了一年多。一年多来，随着各种软件的升级迭代，每次迁移都意味着需要重新配置新的功能，同时也见证了 Git、Hexo 和 NexT 的发展。
>
> 2021.2.10～11，完成了第三次迁移（也许是最后一次迁移），所以打算整理一下一年来搭建博客遇到的各种问题。
>
> 目前已经有许多优秀的博客平台，为什么还要自己搭建博客网站？确实，如果你想纯粹地写博客、分享自己的想法、更容易互相交流，那么博客平台是一个很不错的选择。但是，搭建个人的博客网站可以让你拥有一个只属于你的个人空间，它有极高的自由度，你可以绝对地进行改造，并了解一些关于网站前端的知识。借助于各种优秀的插件，它会变得更加简洁而强大。
>
> 此博客网站为 [Hexo](https://hexo.io/zh-cn/) 框架，依托于 [GitHub Pages](https://pages.github.com) ，使用 [NexT](https://theme-next.js.org) 主题。

# 准备工作

## 安装 Git

- Windows：[点此下载](https://github.com/git-for-windows/git/releases/download/v2.30.1.windows.1/Git-2.30.1-64-bit.exe)（Git for Windows 2.30.1)

- macOS：通过 Homebrew 安装

  ```zsh
  brew install git
  ```

## 安装 Node.js

- Windows：[点此下载](https://nodejs.org/dist/v14.15.5/node-v14.15.5-x86.msi)（node v14.15.5 x86）
- macOS：[点此下载](https://nodejs.org/dist/v14.15.5/node-v14.15.5.pkg)（node v14.15.5）（M1 可通过 Rosetta 2 转译）

## 注册 GitHub

注册 [GitHub](https://github.com) 账号（密码不要太简单），创建名为 `{username}.github.io` 的仓库。

> 假设你的 GitHub 用户名为 `name`，则创建的仓库需命名为 `name.github.io`。

# 配置 Git

## 生成 rsa

Windows 打开 Git Bash，macOS 打开终端（Terminal），检查本机已存在的 ssh 密钥：

```zsh
cd ~/.ssh
```

Windows 若不存在此文件夹，则创建：

```
mkdir ~/.ssh
```

生成 `rsa`（`name@email.com` 为你的 GitHub 的邮件地址）：

```zsh
ssh-keygen -t rsa -C "name@email.com"
```

无需设置密码，所以回车、回车、回车。

执行以下命令：

```zsh
cat id_rsa.pub
```

复制全部（即 `id_rsa.pub` 文件内容）。

## 配置 SSH

打开你的 [GitHub 主页](https://github.com)，点击右上角头像，点击”Settings“进入设置，点击左侧“SSH and GPG keys”，点击右侧“New SSH key”新建密钥。

“Title”随意，将刚才复制的内容粘贴到“Key”处，“Add SSH key”。

此时你可能会收到一封邮件提醒。

本地执行以下命令，测试是否成功：

```zsh
ssh -T git@github.com
```

若提示

```zsh
Are you sure you want to continue connecting (yes/no)?
```

输入 “yes”，回车。

若看到：

```zsh
Hi {username}! You’ve successfully authenticated, but GitHub does not provide shell access.
```

说明 SSH 配置成功。

## 其他配置

配置你的用户名以及 GitHub 邮箱：

```zsh
git config --global user.name "{username}"
git config --global user.email "name@email.com"
```

# 安装 Hexo

执行：

```zsh
npm config set registry https://registry.npm.taobao.org
npm install -g hexo
```

macOS 可能会提示权限问题，尝试许多其他办法无果后，可以尝试 root 执行（不推荐）：

```zsh
sudo su
npm install -g hexo
```

执行：

```zsh
hexo version
```

若正常输出则证明 Hexo 安装成功。

# 初始化本地 Blog

创建一个文件夹，作为博客的根目录。

在此根目录中，执行：

```zsh
hexo init
hexo g
npm install hexo-deployer-git --save
```

修改根目录下 `_config.yml` 文件，将末尾的 `deploy` 部分修改为：

```yaml
deploy:
	type: git
	repository: git@github.com:{username}/{username}.github.io.git
	branch: master
```

执行：

```zsh
hexo d
```

部署博客。

访问 `https://{username}.github.io` ，若出现经典的 `landscape` 页面则说明初始化成功。

# Hexo 基本指令

后续博客文章的生成、发布等操作均通过 Hexo 指令完成。

下面列出一些 Hexo 的基本指令，其他指令请参考 [Hexo 官方文档 - 命令](https://hexo.io/zh-cn/docs/commands)。

## init

```
$ hexo init [folder]
```

新建一个网站。如果没有设置 `folder` ，Hexo 默认在目前的文件夹建立网站。

## new

```
$ hexo new [layout] <title>
```

新建一篇文章。如果没有设置 `layout` 的话，默认使用根目录下 `_config.yml` 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。

```
$ hexo new "post title with whitespace"
```

默认情况下，Hexo 会使用文章的标题来决定文章文件的路径。

## generate

```
$ hexo generate
```

生成静态文件。

| 选项                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `-d`, `--deploy`      | 文件生成后立即部署网站                                       |
| `-w`, `--watch`       | 监视文件变动                                                 |
| `-b`, `--bail`        | 生成过程中如果发生任何未处理的异常则抛出异常                 |
| `-f`, `--force`       | 强制重新生成文件 Hexo 引入了差分机制，如果 `public` 目录存在，那么 `hexo g` 只会重新生成改动的文件。 使用该参数的效果接近 `hexo clean && hexo generate` |
| `-c`, `--concurrency` | 最大同时生成文件的数量，默认无限制                           |

该命令可以简写为

```
$ hexo g
```

## publish

```
$ hexo publish [layout] <filename>
```

发表草稿。

## server

```
$ hexo server
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

| 选项             | 描述                           |
| :--------------- | :----------------------------- |
| `-p`, `--port`   | 重设端口                       |
| `-s`, `--static` | 只使用静态文件                 |
| `-l`, `--log`    | 启动日记记录，使用覆盖记录格式 |

## deploy

```
$ hexo deploy
```

部署网站。

| 参数               | 描述                     |
| :----------------- | :----------------------- |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

该命令可以简写为：

```
$ hexo d
```

## render

```
$ hexo render <file1> [file2] ...
```

渲染文件。

| 参数             | 描述         |
| :--------------- | :----------- |
| `-o`, `--output` | 设置输出路径 |

## migrate

```
$ hexo migrate <type>
```

从其他**博客系统** [迁移内容](https://hexo.io/zh-cn/docs/migration)。

## clean

```
$ hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

## list

```
$ hexo list <type>
```

列出网站资料。

## version

```
$ hexo version
```

显示 Hexo 版本。

# Hexo 基本配置

博客根目录初始化后，文件结构如下：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

其中：

- `package.json` 是应用程序的信息。
- `scaffolds` 是模版文件夹，Hexo 会根据 `scaffold` 来建立文件，是在新建的文章文件中默认填充的内容。
- `source` 是资源文件夹，即存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。
- `themes` 是主题文件夹，Hexo 会根据主题来生成静态页面。

而 `_config.yml` 就是博客网站的配置：

```yaml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site 网站基本信息设置
title: Hexo # 网站的标题
subtitle: '' # 网站的副标题
description: '' # 网站的描述
keywords: # 网站的关键词，支持多个关键词
author: John Doe # 你的名字，同时会出现在主题显示文章的作者
language: en # 网站使用的语言，中国大陆一般设置为 zh-Hans 或 zh-CN，不同主题的要求可能不同
timezone: '' # 网站的时区，中国大陆一般设置为 Asia/Shanghai

# URL URL 设置
## If your site is put in a subdirectory, set url as 'http://example.com/child' and root as '/child/'
url: http://example.com # 网址，以 http:// 或 https:// 开头
root: / # 网站根目录，如果博客不是在上面网站的根目录，可以修改此部分
permalink: :year/:month/:day/:title/ # 文章的永久链接的格式，嫌长可以设置为 :title/，即文章的标题
permalink_defaults: # 永久链接中各部分的默认值
pretty_urls: # 美化 URL
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks 字面意思 我一般设置为 false
  trailing_html: true # Set to false to remove trailing '.html' from permalinks 字面意思 我一般设置为 false

# Directory 目录设置，一般不用改
source_dir: source # 资源文件夹，这个文件夹用来存放内容
public_dir: public # 公共文件夹，这个文件夹用于存放 generate 生成的站点文件
tag_dir: tags # 标签文件夹
archive_dir: archives # 归档文件夹
category_dir: categories # 分类文件夹
code_dir: downloads/code # Include code 文件夹，source_dir 下的子目录
i18n_dir: :lang # 国际化（internationalization，i18n）文件夹
skip_render: # 跳过指定文件的渲染。匹配到的文件将会被不做改动地复制到 public 目录中。你可以使用 glob 表达式来匹配路径：如 "mypage/**/*" 和 "_post/test-post.md"。如果文件少你也可以通过在对应文件开头加上“layout: false来屏蔽渲染

# Writing 文章设置
new_post_name: :title.md # File name of new posts 新文章的文件名
default_layout: post # 预设布局，前面说过
titlecase: false # Transform title into titlecase 把标题转换为 title case
external_link: # 在新标签页中打开链接
  enable: true # Open external links in new tab 在新标签页中打开链接
  field: site # Apply to the whole site 对整个网站（site）生效或仅对文章（post）生效
  exclude: '' # 需要排除的域名。主域名和子域名如 www 需分别配置
filename_case: 0 # 把文件名称转换为 (1) 小写或 (2) 大写
render_drafts: false # 显示草稿
post_asset_folder: false # 新建文章 md 的时候是否再新建一个同名文件夹用于存放图片等引用文件
relative_link: false # 把链接改为与根目录的相对位址
future: true # 显示未来的文章
highlight: # Highlight.js 代码块的设置
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false
prismjs: # PrismJS 代码块的设置
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''

# 默认情况下，Hexo 生成的超链接都是绝对地址。
# 例如，如果您的网站域名为 example.com,您有一篇文章名为 hello，那么绝对链接可能像这样：
# http://example.com/hello.html，它是绝对于域名的。
# 相对链接像这样：/hello.html，
# 也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。
# 通常情况下，建议使用绝对地址。

# Home page setting 不用改
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date

# Category & Tag 分类和标签，不用改
default_category: uncategorized # 默认分类
category_map: # 分类别名
tag_map: # 标签别名

# Metadata elements 不用改
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true

# Date / Time format 日期时间格式，不用改
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD # 日期格式
time_format: HH:mm:ss # 时间格式
## updated_option supports 'mtime', 'date', 'empty'
updated_option: 'mtime' # 当 Front Matter 中没有指定 updated 时 updated 的取值
# mtime: 使用文件的最后修改时间。这是从 Hexo 3.0.0 开始的默认行为。
# date: 使用 date 作为 updated 的值。可被用于 Git 工作流之中，因为使用 Git 管理站点时，文件的最后修改日期常常会发生改变
# empty: 直接删除 updated。使用这一选项可能会导致大部分主题和插件无法正常工作。
# use_date_for_updated 选项已经被废弃，将会在下个重大版本发布时去除。请改为使用 updated_option: 'date'。

# Pagination 分页，不用改
## Set per_page to 0 to disable pagination
per_page: 10 # 每页显示的文章量 (0 = 关闭分页功能)
pagination_dir: page # 分页目录

# Include / Exclude file(s) 包括或不包括目录和文件，让 Hexo 进行处理或忽略某些目录和文件夹。你可以使用 glob 表达式对目录和文件进行匹配。
## include:/exclude: options only apply to the 'source/' folder
include: # Hexo 默认会忽略隐藏文件和文件夹（包括名称以下划线和 . 开头的文件和文件夹，Hexo 的 _posts 和 _data 等目录除外）。通过设置此字段将使 Hexo 处理他们并将它们复制到 source 目录下。
exclude: # Hexo 会忽略这些文件和目录
ignore: # Ignore files/folders

# Extensions 扩展
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape # 当前主题名称（主题文件夹下特定主题文件夹的名字），false 则禁用主题

# Deployment 部署设置，必须更改，前面说了
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type:


```

今年迁移博客的时候发现，Hexo 升级至 5.0.0 以上后，新增加了独立的 `_config.[theme].yml` 文件，在站点根目录下，支持 `yml` 或 `json` 格式。需要配置站点 `_config.yml` 文件中的 `theme` 以供 Hexo 寻找 `_config.[theme].yml` 文件。

> 我们强烈建议你将所有的主题配置集中在一处。如果你不得不在多处配置你的主题，那么这些信息对你将会非常有用：Hexo 在合并主题配置时，Hexo 配置文件中的 `theme_config` 的优先级最高，其次是 `_config.[theme].yml` 文件，最后是位于主题目录下的 `_config.yml` 文件。

# 安装 NexT

经过上述操作，基本上已经配置完成了。如果追求美观以及其他高级功能，可以继续看。

Hexo 的默认主题是 landscape，推荐使用 [NexT 主题](https://theme-next.js.org)。

![](https://theme-next.js.org/images/next-schemes-dark.png)

如果搜索 NexT 主题，会发现在 GitHub 上有三个 NexT 主题的仓库：

- 一个是 v6.0.0 之前版本的[个人仓库](https://github.com/iissnan/hexo-theme-next)，主题原作者停止维护。
- 后来有些人单独创立了一个名为 [theme-next](https://github.com/theme-next/) 的团队，v6.0.0 至 v7.8.0 版本主题在这个[仓库](https://github.com/theme-next/hexo-theme-next)中。然而，因 theme-next 团队管理者自 2019 年 10 月起长期不在线，管理者也没有给其他成员足够的权限，导致仓库管理出现很多问题。
- 自 2020 年 5 月起，theme-next 团队部分成员迁移至 [next-theme](https://github.com/next-theme/)，新团队开发的 NexT 版本为 v8.0.0 版本。

因此目前 NexT 主题共有三个仓库：

- 2014-2017：https://github.com/iissnan/hexo-theme-next
- 2018-2019：https://github.com/theme-next/hexo-theme-next
- 2020-：https://github.com/next-theme/hexo-theme-next

建议下载并安装最新版本：

```zsh
npm install hexo-theme-next
```

> 注：git clone 下来的代码可能为旧版本（7.8.0）。

此时博客根目录下的 `theme` 文件夹内增加了 `next` 主题文件夹，将“站点配置文件”的 `theme` 改为 `next` ，安装完成。

# NexT 基本配置

上面的 Hexo 基本配置部分已经分析了“站点配置文件”，同理配置 NexT 主题需要修改“主题配置文件”，即 `next` 文件夹下的 `_config.yml` 。

这是传统的方式。最新版的 NexT 和 Hexo 支持“Alternate Theme Config”，即通过修改根目录下的 `_config.[name].yml` （`_config.next.yml`）文件来配置主题。Hexo 官方已实现此功能，在升级到 Hexo 5.0 版本后，请留意配置方式上的改变，使用 `_config.next.yml` 代替 `source/_data/next.yml`。旧的 `next.yml` 配置方式诞生于 2015 年（[iissnan/hexo-theme-next#328](https://github.com/iissnan/hexo-theme-next/issues/328)），已经完成其历史使命，将在 NexT v8.1.0 版本后停止支持。详细请参考 [NexT 官方文档 1](https://theme-next.js.org/docs/getting-started/configuration.html)、[NexT 官方文档 2](https://theme-next.js.org/docs/getting-started/configuration.html)。

打开“主题配置文件”，参考自 [NexT 官方文档 - 配置](https://theme-next.js.org/docs/theme-settings/sidebar.html)：

```yaml
# ===============================================================
# It's recommended to use Alternate Theme Config to configure NexT
# Modifying this file may result in merge conflict
# See: https://theme-next.js.org/docs/getting-started/configuration
# ===============================================================

# ---------------------------------------------------------------
# Theme Core Configuration Settings
# See: https://theme-next.js.org/docs/theme-settings/
# ---------------------------------------------------------------

# Allow to cache content generation. 缓存内容生成
cache:
  enable: true

# Remove unnecessary files after hexo generate. 移除不需要的文件
minify: false

# Define custom file paths. 自定义样式布局，你可以对以下十个部分进行私人定制。这是新版本的新功能，旧版本需要自行修改原有的主题内部文件，所以搜索引擎查到的有些高级功能的实现还是“老方法”。更新此功能更易于管理，且不易出错。配合 Chrome 的调试功能，可以实现对界面的完全改造。NexT 建议使用新方法，不提倡修改主题内部文件的旧方法，请谨慎甄别，因为其内容可能过时。
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  #footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  #style: source/_data/styles.styl


# ---------------------------------------------------------------
# Scheme Settings 主题方案设置
# ---------------------------------------------------------------

# Schemes 选一个方案，我的是 Mist
scheme: Muse
#scheme: Mist
#scheme: Pisces
#scheme: Gemini

# Dark Mode 深色模式，auto 为自动，true 为常开
darkmode: false


# ---------------------------------------------------------------
# Site Information Settings 网站信息设置
# ---------------------------------------------------------------

favicon: # 网站的图标设置，文件在主题文件夹 source 里
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json

# Custom Logo (Warning: Do not support scheme Mist)
custom_logo: #/uploads/custom-logo.jpg

# Creative Commons 4.0 International License.
# See: https://creativecommons.org/about/cclicenses/
# Available values of license: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | zero
# You can set a language value if you prefer a translated version of CC license, e.g. deed.zh
# CC licenses are available in 39 languages, you can find the specific and correct abbreviation you need on https://creativecommons.org
creative_commons:
  license: by-nc-sa
  sidebar: false
  post: false
  language:


# ---------------------------------------------------------------
# Menu Settings 菜单设置
# ---------------------------------------------------------------

# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If the translation for this item is available, the translated text will be loaded, otherwise the Key name will be used. Key is case-senstive.
# Value before `||` delimiter is the target link, value after `||` delimiter is the name of Font Awesome icon.
# External url should start with http:// or https://
menu: # 一级目录设置，多级目录不常用，其设置见官方文档
  #home: / || fa fa-home # 主界面菜单，某一旧版本在此处出现了 bug，原因是/和||之间的空格，新版本只需注意图标名称与旧版本不同，下面的同理
  #about: /about/ || fa fa-user # 关于菜单
  #tags: /tags/ || fa fa-tags # 标签菜单，如果文章带有标签，建议打开，并 hexo new page tags，在根目录下的 source 文件夹中生成了对应名称的文件夹，在其中的 index.md 添加 type: "tags"，下面的同理
  #categories: /categories/ || fa fa-th # 分类菜单
  #archives: /archives/ || fa fa-archive # 归档菜单，可以打开
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat

# Enable / Disable menu icons / item badges. 图标
menu_settings:
  icons: true
  badges: false


# ---------------------------------------------------------------
# Sidebar Settings 侧边栏设置
# See: https://theme-next.js.org/docs/theme-settings/sidebar
# ---------------------------------------------------------------

sidebar:
  # Sidebar Position. 侧边栏位置
  position: left
  #position: right

  # Manual define the sidebar width. If commented, will be default for:
  # Muse | Mist: 320
  # Pisces | Gemini: 240
  #width: 300

  # Sidebar Display (only for Muse | Mist), available values:
  #  - post    expand on posts automatically. Default. 在文章中打开侧边栏
  #  - always  expand for all pages automatically. 常开
  #  - hide    expand only when click on the sidebar toggle icon. 常隐藏
  #  - remove  totally remove sidebar including sidebar toggle. 移除
  display: post

  # Sidebar padding in pixels. 不用改
  padding: 18
  # Sidebar offset from top menubar in pixels (only for Pisces | Gemini).
  offset: 12
  # Enable sidebar on narrow view (only for Muse | Mist). 窄屏幕（手机）是否显示侧边栏
  onmobile: false

# Sidebar Avatar 侧边栏头像，和上面的头像一样，注意文件的路径和名称，还有扩展名
avatar:
  # Replace the default image and set the url here.
  url: #/images/avatar.gif
  # If true, the avatar will be dispalyed in circle. 圆的？
  rounded: false
  # If true, the avatar will be rotated with the cursor. 旋转的？（焦点移到上面会旋转）
  rotated: false

# Posts / Categories / Tags in sidebar.
site_state: true

# Social Links 社交账号
# Usage: `Key: permalink || icon`
# Key is the link label showing to end users.
# Value before `||` delimiter is the target permalink, value after `||` delimiter is the name of Font Awesome icon.
social:
  #GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
  #Weibo: https://weibo.com/yourname || fab fa-weibo
  #Google: https://plus.google.com/yourname || fab fa-google
  #Twitter: https://twitter.com/yourname || fab fa-twitter
  #FB Page: https://www.facebook.com/yourname || fab fa-facebook
  #StackOverflow: https://stackoverflow.com/yourname || fab fa-stack-overflow
  #YouTube: https://youtube.com/yourname || fab fa-youtube
  #Instagram: https://instagram.com/yourname || fab fa-instagram
  #Skype: skype:yourname?call|chat || fab fa-skype

social_icons:
  enable: true
  icons_only: false
  transition: false

# Blog rolls
links_settings:
  icon: fa fa-globe
  title: Links
  # Available values: block | inline
  layout: block

links: # 链接，可以用来添加友链
  #Title: http://yoursite.com

# Table of Contents in the Sidebar 侧边栏中的文章目录
# Front-matter variable (unsupport wrap expand_all).
toc:
  enable: true
  # Automatically add list number to toc. 是否编号
  number: true
  # If true, all words will placed on next lines if header width longer then sidebar width. 标题字数太多怎么办
  wrap: false
  # If true, all level of TOC in a post will be displayed, rather than the activated part of it. 次级标题是否一直展开
  expand_all: false
  # Maximum heading depth of generated toc. 标题最大层数
  max_depth: 6

# A button to open designated chat widget in sidebar. 聊天？
# Firstly, you need enable the chat service you want to activate its sidebar button.
chat:
  enable: false
  #service: chatra
  #service: tidio
  icon: fa fa-comment # Icon name in Font Awesome, set false to disable icon.
  text: Chat # Button text, change it as you wish.


# ---------------------------------------------------------------
# Footer Settings 页脚设置，8.0 新增
# See: https://theme-next.js.org/docs/theme-settings/footer
# ---------------------------------------------------------------

# Show multilingual switcher in footer.
language_switcher: false # 页脚的语言切换

footer: # 页脚设置
  # Specify the date when the site was setup. If not defined, current year will be used.
  #since: 2020 # 网站建立日期

  # Icon between year and copyright info. since 和版权之间的图标
  icon:
    # Icon name in Font Awesome. See: https://fontawesome.com/icons
    name: fa fa-heart # Font Awesome 中的图标名，这一点和旧版本不一样
    # If you want to animate the icon, set it to true.
    animated: false # 动态效果，某一版本新增的功能
    # Change the color of icon, using Hex Code.
    color: "#ff0000" # 图标颜色

  # If not defined, `author` from Hexo `_config.yml` will be used.
  copyright: # 显示版权的名字

  # Powered by Hexo & NexT
  powered: true # 是否显示 Powered by Hexo & NexT

  # Beian ICP and gongan information for Chinese users. See: https://beian.miit.gov.cn, http://www.beian.gov.cn 备案信息
  beian:
    enable: false
    icp:
    # The digit in the num of gongan beian.
    gongan_id:
    # The full num of gongan beian.
    gongan_num:
    # The icon for gongan beian. See: http://www.beian.gov.cn/portal/download
    gongan_icon_url:


# ---------------------------------------------------------------
# Post Settings 文章发布设置
# See: https://theme-next.js.org/docs/theme-settings/posts
# ---------------------------------------------------------------

# Automatically excerpt description in homepage as preamble text. 文章简介
excerpt_description: true

# Read more button 阅读更多按钮
# If true, the read more button will be displayed in excerpt section.
read_more_btn: true

# Post meta display settings 文章元数据，即每篇文章的基本信息
post_meta:
  item_text: true
  created_at: true
  updated_at:
    enable: true
    another_day: true
  categories: true

# Post wordcount display settings 字数统计
# Dependencies: https://github.com/next-theme/hexo-word-counter 依赖的插件，具体参考插件说明
symbols_count_time:
  separated_meta: true
  item_text_total: false

# Use icon instead of the symbol # to indicate the tag at the bottom of the post 显示标签用图标还是 # 号
tag_icon: false

# Donate (Sponsor) settings 要钱
# Front-matter variable (unsupport animation).
reward_settings:
  # If true, a donate button will be displayed in every article by default.
  enable: false
  animation: false
  #comment: Buy me a coffee 要钱时说什么

reward: # 要钱方式，注意文件路径、文件名和扩展名
  #wechatpay: /images/wechatpay.png
  #alipay: /images/alipay.png
  #paypal: /images/paypal.png
  #bitcoin: /images/bitcoin.png

# Subscribe through Telegram Channel, Twitter, etc. 诱导关注
# Usage: `Key: permalink || icon` (Font Awesome)
follow_me:
  #Twitter: https://twitter.com/username || fab fa-twitter
  #Telegram: https://t.me/channel_name || fab fa-telegram
  #WeChat: /images/wechat_channel.jpg || fab fa-weixin
  #RSS: /atom.xml || fa fa-rss

# Related popular posts 相关文章
# Dependencies: https://github.com/tea3/hexo-related-popular-posts 依赖的插件
related_posts:
  enable: false
  title: # Custom header, leave empty to use the default one
  display_in_home: false
  params:
    maxCount: 5
    #PPMixingRate: 0.0
    #isDate: false
    #isImage: false
    #isExcerpt: false

# Post edit 文章在线编辑
# Easily browse and edit blog source code online.
post_edit:
  enable: false
  url: https://github.com/user-name/repo-name/tree/branch-name/subdirectory-name # Link for view source
  #url: https://github.com/user-name/repo-name/edit/branch-name/subdirectory-name # Link for fork & edit

# Show previous post and next post in post footer if exists
# Available values: left | right | false
post_navigation: left


# ---------------------------------------------------------------
# Custom Page Settings 个性化设置
# See: https://theme-next.js.org/docs/theme-settings/custom-pages
# ---------------------------------------------------------------

# TagCloud settings for tags page. 标签云
tagcloud:
  min: 12 # Minimun font size in px 最小的
  max: 30 # Maxium font size in px 最大的
  amount: 200 # Total amount of tags 最多有多少
  orderby: name # Order of tags
  order: 1 # Sort order

# Google Calendar 谷歌日历
# Share your recent schedule to others via calendar page.
calendar:
  calendar_id: <required> # Your Google account E-Mail
  api_key: <required>
  orderBy: startTime
  offsetMax: 24 # Time Range
  offsetMin: 4 # Time Range
  showDeleted: false
  singleEvents: true
  maxResults: 250


# ---------------------------------------------------------------
# Misc Theme Settings 其他主题设置
# See: https://theme-next.js.org/docs/theme-settings/miscellaneous
# ---------------------------------------------------------------

# Preconnect CDN for fonts and plugins.
# For more information: https://www.w3.org/TR/resource-hints/#preconnect
preconnect: false

# Set the text alignment in posts / pages. 不用改
text_align:
  # Available values: start | end | left | right | center | justify | justify-all | match-parent
  desktop: justify
  mobile: justify

# Reduce padding / margin indents on devices with narrow width. 不用改
mobile_layout_economy: false

# Android Chrome header panel color ($brand-bg / $headband-bg => $black-deep).
android_chrome_color: "#222"

codeblock:
  # Code Highlight theme 代码块主题
  # All available themes: https://theme-next.js.org/highlight/
  theme:
    light: default
    dark: tomorrow-night
  prism:
    light: prism
    dark: prism-dark
  # Add copy button on codeblock 代码块复制按钮
  copy_button:
    enable: false
    # Available values: default | flat | mac 风格
    style:

back2top: # 回顶键
  enable: true
  # Back to top in sidebar.
  sidebar: false
  # Scroll percent label in b2t button.
  scrollpercent: false

# Reading progress bar 页面浏览进度条
reading_progress:
  enable: false
  # Available values: top | bottom
  position: top
  color: "#37c6c0"
  height: 3px

# Bookmark Support 书签，记录页面浏览进度
bookmark:
  enable: false
  # Customize the color of the bookmark.
  color: "#222"
  # If auto, save the reading progress when closing the page or clicking the bookmark-icon.
  # If manual, only save it by clicking the bookmark-icon.
  save: auto

# `Follow me on GitHub` banner in the top-right corner. 右上角 GitHub 图标
github_banner:
  enable: false
  permalink: https://github.com/yourname
  title: Follow me on GitHub


# ---------------------------------------------------------------
# Font Settings 字体设置
# ---------------------------------------------------------------
# Find fonts on Google Fonts (https://www.google.com/fonts)
# All fonts set here will have the following styles:
#   light | light italic | normal | normal italic | bold | bold italic
# Be aware that setting too much fonts will cause site running slowly
# ---------------------------------------------------------------
# Web Safe fonts are recommended for `global` (and `title`):
# Arial | Tahoma | Helvetica | Times New Roman | Courier New | Verdana | Georgia | Palatino | Garamond | Comic Sans MS | Trebuchet MS
# ---------------------------------------------------------------

font:
  enable: false

  # Uri of fonts host, e.g. https://fonts.googleapis.com (Default).
  host:

  # Font options:
  # `external: true` will load this font family from `host` above.
  # `family: Times New Roman`. Without any quotes.
  # `size: x.x`. Use `em` as unit. Default: 1 (16px)

  # Global font settings used for all elements inside <body>.
  global:
    external: true
    family: Lato
    size:

  # Font settings for site title (.site-title).
  title:
    external: true
    family:
    size:

  # Font settings for headlines (<h1> to <h6>).
  headings:
    external: true
    family:
    size:

  # Font settings for posts (.post-body).
  posts:
    external: true
    family:

  # Font settings for <code> and code blocks.
  codes:
    external: true
    family:


# ---------------------------------------------------------------
# SEO Settings 搜索引擎优化
# See: https://theme-next.js.org/docs/theme-settings/seo
# ---------------------------------------------------------------

# If true, site-subtitle will be added to index page. 浏览器标签页是否显示网站副标题
# Remember to set up your site-subtitle in Hexo `_config.yml` (e.g. subtitle: Subtitle)
index_with_subtitle: false

# Automatically add external URL with Base64 encrypt & decrypt. 不用改
exturl: false

# Google Webmaster tools verification.
# See: https://www.google.com/webmasters
google_site_verification:

# Bing Webmaster tools verification.
# See: https://www.bing.com/webmaster
bing_site_verification:

# Yandex Webmaster tools verification.
# See: https://webmaster.yandex.ru
yandex_site_verification:

# Baidu Webmaster tools verification.
# See: https://ziyuan.baidu.com/site
baidu_site_verification:


# ---------------------------------------------------------------
# Third Party Plugins & Services Settings 第三方插件及服务设置
# See: https://theme-next.js.org/docs/third-party-services/
# More plugins: https://github.com/next-theme/awesome-next
# You may need to install the corresponding dependency packages
# ---------------------------------------------------------------

# Math Formulas Render Support 数学公式
# Warning: Please install / uninstall the relevant renderer according to the documentation.
# See: https://theme-next.js.org/docs/third-party-services/math-equations
# Server-side plugin: https://github.com/next-theme/hexo-filter-mathjax
math:
  # Default (false) will load mathjax / katex script on demand.
  # That is it only render those page which has `mathjax: true` in Front-matter. 字面意思，关闭则意味着是否渲染公式依赖于每篇文章的设置
  # If you set it to true, it will load mathjax / katex srcipt EVERY PAGE.
  every_page: true

  mathjax:
    enable: false
    # Available values: none | ams | all
    tags: none

  katex:
    enable: false
    # See: https://github.com/KaTeX/KaTeX/tree/master/contrib/copy-tex
    copy_tex: false

# Easily enable fast Ajax navigation on your website. 跳转时不加载重复元素，更快跳转，但可能会出现 bug
# For more information: https://github.com/next-theme/pjax
pjax: false

# FancyBox is a tool that offers a nice and elegant way to add zooming functionality for images. 放大文章中的图片，和下面的只能开启一个
# For more information: https://fancyapps.com/fancybox
fancybox: false

# A JavaScript library for zooming images like Medium. 放大文章中的图片，和上面的只能开启一个
# Warning: Do not enable both `fancybox` and `mediumzoom`.
# For more information: https://medium-zoom.francoischalifour.com
mediumzoom: false

# Vanilla JavaScript plugin for lazyloading images. 懒加载
# For more information: https://apoorv.pro/lozad.js/demo/
lazyload: false

# Pangu Support
# For more information: https://github.com/vinta/pangu.js
# Server-side plugin: https://github.com/next-theme/hexo-pangu
pangu: false

# Quicklink Support
# For more information: https://getquick.link
# Front-matter variable (unsupport home archive).
quicklink:
  enable: false

  # Home page and archive page can be controlled through home and archive options below.
  # This configuration item is independent of `enable`.
  home: false
  archive: false

  # Default (true) will initialize quicklink after the load event fires.
  delay: true
  # Custom a time in milliseconds by which the browser must execute prefetching.
  timeout: 3000
  # Default (true) will attempt to use the fetch() API if supported (rather than link[rel=prefetch]).
  priority: true

  # For more flexibility you can add some patterns (RegExp, Function, or Array) to ignores.
  # See: https://github.com/GoogleChromeLabs/quicklink#custom-ignore-patterns
  ignores:


# ---------------------------------------------------------------
# Comments Settings 评论设置
# See: https://theme-next.js.org/docs/third-party-services/comments
# ---------------------------------------------------------------

# Multiple Comment System Support
comments:
  # Available values: tabs | buttons
  style: tabs
  # Choose a comment system to be displayed by default.
  # Available values: disqus | disqusjs | changyan | livere | gitalk | utterances
  active:
  # Setting `true` means remembering the comment system selected by the visitor.
  storage: true
  # Lazyload all comment systems.
  lazyload: false
  # Modify texts or order for any navs, here are some examples.
  nav:
    #disqus:
    #  text: Load Disqus
    #  order: -1
    #gitalk:
    #  order: -2

# Disqus
disqus:
  enable: false
  shortname:
  count: true

# DisqusJS
# For more information: https://disqusjs.skk.moe
disqusjs:
  enable: false
  # API Endpoint of Disqus API (https://disqus.com/api/docs).
  # Leave api empty if you are able to connect to Disqus API. Otherwise you need a reverse proxy for it.
  # For example:
  # api: https://disqus.skk.moe/disqus/
  api:
  apikey: # Register new application from https://disqus.com/api/applications/
  shortname: # See: https://disqus.com/admin/settings/general/

# Changyan
changyan:
  enable: false
  appid:
  appkey:

# LiveRe comments system
# You can get your uid from https://livere.com/insight/myCode (General web site)
livere_uid: # <your_uid>

# Gitalk
# For more information: https://gitalk.github.io
gitalk:
  enable: false
  github_id: # GitHub repo owner
  repo: # Repository name to store issues
  client_id: # GitHub Application Client ID
  client_secret: # GitHub Application Client Secret
  admin_user: # GitHub repo owner and collaborators, only these guys can initialize gitHub issues
  distraction_free_mode: true # Facebook-like distraction free mode
  # Gitalk's display language depends on user's browser or system environment
  # If you want everyone visiting your site to see a uniform language, you can set a force language value
  # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
  language:

# Utterances
# For more information: https://utteranc.es
utterances:
  enable: false
  repo: # Github repository name
  # Available values: pathname | url | title | og:title
  issue_term: pathname
  # Available values: github-light | github-dark | preferred-color-scheme | github-dark-orange | icy-dark | dark-blue | photon-dark | boxy-light
  theme: github-light

# Isso
# For more information: https://posativ.org/isso/
isso: # <data_isso>


# ---------------------------------------------------------------
# Post Widgets & Content Sharing Services
# See: https://theme-next.js.org/docs/third-party-services/post-widgets
# ---------------------------------------------------------------

# Star rating support to each article.
# To get your ID visit https://widgetpack.com
rating:
  enable: false
  id:     # <app_id>
  color:  "#fc6423"

# AddThis Share. See: https://www.addthis.com
# Go to https://www.addthis.com/dashboard to customize your tools.
add_this_id:


# ---------------------------------------------------------------
# Statistics and Analytics
# See: https://theme-next.js.org/docs/third-party-services/statistics-and-analytics
# ---------------------------------------------------------------

# Google Analytics 谷歌分析
# See: https://analytics.google.com
google_analytics:
  tracking_id: # <app_id>
  # By default, NexT will load an external gtag.js script on your site.
  # If you only need the pageview feature, set the following option to true to get a better performance.
  only_pageview: false

# Baidu Analytics 百度统计
# See: https://tongji.baidu.com
baidu_analytics: # <app_id>

# Growingio Analytics
# See: https://www.growingio.com
growingio_analytics: # <project_id>

# Cloudflare Web Analytics
# See: https://www.cloudflare.com/web-analytics/
cloudflare_analytics:

# Show number of visitors of each article. 访客统计 leancloud
# You can visit https://www.leancloud.cn to get AppID and AppKey.
leancloud_visitors:
  enable: false
  app_id: # <your app id>
  app_key: # <your app key>
  # Required for apps from CN region
  server_url: # <your server url>
  # Dependencies: https://github.com/theme-next/hexo-leancloud-counter-security
  # If you don't care about security in leancloud counter and just want to use it directly
  # (without hexo-leancloud-counter-security plugin), set `security` to `false`.
  security: true

# Another tool to show number of visitors to each article.
# Visit https://console.firebase.google.com/u/0/ to get apiKey and projectId.
# Visit https://firebase.google.com/docs/firestore/ to get more information about firestore.
firestore:
  enable: false
  collection: articles # Required, a string collection name to access firestore database
  apiKey: # Required
  projectId: # Required

# Show Views / Visitors of the website / page with busuanzi. 访客统计 不蒜子
# For more information: http://ibruce.info/2015/04/04/busuanzi
busuanzi_count:
  enable: false
  total_visitors: true
  total_visitors_icon: fa fa-user
  total_views: true
  total_views_icon: fa fa-eye
  post_views: true
  post_views_icon: fa fa-eye


# ---------------------------------------------------------------
# Search Services 搜索服务
# See: https://theme-next.js.org/docs/third-party-services/search-services
# ---------------------------------------------------------------

# Algolia Search
# For more information: https://www.algolia.com
algolia_search:
  enable: false
  hits:
    per_page: 10

# Local Search
# Dependencies: https://github.com/next-theme/hexo-generator-searchdb
local_search:
  enable: false
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false


# ---------------------------------------------------------------
# Chat Services 聊天服务
# See: https://theme-next.js.org/docs/third-party-services/chat-services
# ---------------------------------------------------------------

# Chatra Support
# For more information: https://chatra.com
# Dashboard: https://app.chatra.io/settings/general
chatra:
  enable: false
  async: true
  id: # Visit Dashboard to get your ChatraID
  #embed: # Unfinished experimental feature for developers. See: https://chatra.com/help/api/#injectto

# Tidio Support
# For more information: https://www.tidio.com
# Dashboard: https://www.tidio.com/panel/dashboard
tidio:
  enable: false
  key: # Public Key, get it from dashboard. See: https://www.tidio.com/panel/settings/developer


# ---------------------------------------------------------------
# Tags Settings 标签服务，特殊的标签，访问下面的官网了解
# See: https://theme-next.js.org/docs/tag-plugins/
# ---------------------------------------------------------------

# Note tag (bootstrap callout)
note:
  # Note tag style values:
  #  - simple    bootstrap callout old alert style. Default.
  #  - modern    bootstrap callout new (v2-v3) alert style.
  #  - flat      flat callout style with background, like on Mozilla or StackOverflow.
  #  - disabled  disable all CSS styles import of note tag.
  style: simple
  icons: false # 可以打开
  # Offset lighter of background in % for modern and flat styles (modern: -12 | 12; flat: -18 | 6).
  # Offset also applied to label tag variables. This option can work with disabled note tag.
  light_bg_offset: 0

# Tabs tag
tabs:
  transition:
    tabs: false
    labels: true

# PDF tag
# NexT will try to load pdf files natively, if failed, pdf.js will be used.
# So, you have to install the dependency of pdf.js if you want to use pdf tag and make it available to all browsers.
# Dependencies: https://github.com/next-theme/theme-next-pdf
pdf:
  enable: false
  # Default height
  height: 500px

# Mermaid tag
mermaid:
  enable: false
  # Available themes: default | dark | forest | neutral
  theme: forest


# ---------------------------------------------------------------
# Animation Settings 动态效果设置
# ---------------------------------------------------------------

# Use Animate.css to animate everything.
# For more information: https://animate.style
motion:
  enable: true
  async: false
  transition:
    # All available transition variants: https://theme-next.js.org/animate/
    post_block: fadeIn
    post_header: fadeInDown
    post_body: fadeInDown
    coll_header: fadeInLeft
    # Only for Pisces | Gemini.
    sidebar: fadeInUp

# Progress bar in the top during page loading.
# For more information: https://github.com/rstacruz/nprogress
nprogress:
  enable: false
  spinner: true

# Canvas ribbon
# For more information: https://github.com/hustcc/ribbon.js
canvas_ribbon:
  enable: false
  size: 300 # The width of the ribbon
  alpha: 0.6 # The transparency of the ribbon
  zIndex: -1 # The display level of the ribbon


#! ==============================================================
#! DO NOT EDIT THE FOLLOWING SETTINGS
#! UNLESS YOU KNOW WHAT YOU ARE DOING
#! See: https://theme-next.js.org/docs/advanced-settings/vendors
#! ==============================================================

# It's recommended to use the same version as in `_vendors.yml` to avoid potential problems.
# Remember to use the HTTPS protocol of CDN links when you enable HTTPS on your site.
vendors:
  # The CDN provider of NexT internal scripts.
  # Available values: local | jsdelivr | unpkg | cdnjs
  # Warning: If you are using the latest master branch of NexT, please set `internal: local`
  internal: local
  # The default CDN provider of third-party plugins.
  # Available values: local | jsdelivr | unpkg | cdnjs
  # Dependencies for `plugins: local`: https://github.com/next-theme/plugins
  plugins: jsdelivr

  # In the following settings, you can specify the CDN link for each plugin.
  # If left blank, the default CDN provider set by `plugins` option will be used.

  # Anime.js
  # For more information: https://animejs.com
  anime:

  # Font Awesome
  # For more information: https://fontawesome.com
  fontawesome:

  # Prism
  prism:
  prism_autoloader:
  prism_line_numbers:

  # MathJax
  mathjax:

  # KaTeX
  katex:
  copy_tex_js:
  copy_tex_css:

  # Pjax
  pjax:

  # FancyBox
  jquery:
  fancybox_js:
  fancybox_css:

  # Medium-zoom
  mediumzoom:

  # Lazyload
  lazyload:

  # Pangu
  pangu:

  # Quicklink
  quicklink:

  # DisqusJS
  disqusjs_js:
  disqusjs_css:

  # Gitalk
  gitalk_js:
  gitalk_css:

  # Firebase
  firebase_app:
  firebase_firestore:

  # Algolia Search
  algolia_search:
  instant_search:

  # PDF
  pdfobject:

  # Mermaid
  mermaid:

  # Animate.css
  # Warning: motion won't work with animate.css version 3.2.0 or later
  animate_css:

  # NProgress.js
  nprogress_js:
  nprogress_css:

  # Canvas ribbon
  canvas_ribbon:

# Assets
# Accelerate delivery of static files using a CDN
css: css
js: js
images: images

```

上述配置就是 NexT 主题提供的基本配置。按照你的喜好修改之后，博客网站也就大体成型了。

如果你不满足于此，想要添加更多的功能，可以看 [下一篇文章](https://superpung.com/blog-config-1/) 。
