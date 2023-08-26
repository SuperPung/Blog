---
url: blog-top-article
title: 博客文章置顶办法
date: 2021-02-17 14:09:11
categories: [技术]
tags: [博客配置]
---

如果你想让访客访问你的博客时首先看到某些文章，置顶功能必不可少。

<!--more-->

这一功能没找到在主题中实现的方法，只能修改 Hexo 文件。

修改 `/node_modules/hexo-generator-index/lib/generator.js` 文件为：

```javascript
'use strict';

const pagination = require('hexo-pagination');
const { sort } = require('timsort');

module.exports = function(locals) {
  const config = this.config;
  const posts = locals.posts.sort(config.index_generator.order_by);

  // top
  posts.data = posts.data.sort(function(a, b) {
      if(a.top && b.top) {
          if(a.top == b.top) return b.date - a.date;
          else return b.top - a.top;
      }
      else if(a.top && !b.top) {
          return -1;
      }
      else if(!a.top && b.top) {
          return 1;
      }
      else return b.date - a.date;
  });

  sort(posts.data, (a, b) => (b.sticky || 0) - (a.sticky || 0));

  const paginationDir = config.pagination_dir || 'page';
  const path = config.index_generator.path || '';

  return pagination(path, posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

即通过比较 `top` 值来决定文章的排列顺序。

这样添加完成之后，置顶文章并没有明显的标志。此时可以通过开启 `custom_file_path` 中的 `postMeta`，并在 `/source/_data/` 目录下新建 `post-meta.njk` 文件来进一步操作，但是效果是这样的：

![错误top](https://i0.hdslb.com/bfs/album/1d74f9997818ec2620e8be281feac60b08bf7dbb.png)

要想把图标放在左侧，只能通过“不推荐”的方法修改主题文件来实现这种效果：

![正确top](https://i0.hdslb.com/bfs/album/cc4e1ec6ca511f8a7bc44ee551d983f344feee35.png)

修改 `/themes/next/layout/_partials/post/post-meta.njk` 文件，在 `<div class="post-meta">` 下添加以下代码：

```html
  {% if post.top %}
    <span class="post-top-icon">
      <i class="fas fa-thumbtack"></i>
    </span>
    <font color=FF0000>&nbsp;TOP</font>
  {% endif %}
```

在 NexT 旧版本中，要修改的是 `/themes/next/layout/_macro/post.swig` 文件。新版本的 NexT 对原 `post` 文件做了分离。在 Font Awesome 旧版本中，对应的大头针图标名称为 `fa fa-thumb-tack`，新版改为 `fas fa-thumbtack`。另外，`color` 为显示字体的颜色，可以更改为你想要的颜色。后面的提示文字也可以更改为“置顶”等。

完成。

> 参考
>
> - [解决 Hexo 置顶问题 | Netcan on Programming](https://netcan.github.io/2015/11/22/解决Hexo置顶问题/)
> - [Hexo博客彻底解决置顶问题 | wangwlj's Blog](http://wangwlj.com/2018/01/09/blog_pin_post/)
