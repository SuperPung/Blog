---
url: blog-insert-images
title: 在博客文章中插入图片
date: 2021-02-17 14:26:19
categories: [技术]
tags: [博客配置]
---

“插入图片”会遇到什么问题呢？

<!--more-->

“插入图片”本身就有两种含义：

- 一种是在 Markdown 文档中插入图片，只需要 `![图片标题](图片路径)` 就可以；
- 另一种是让 Hexo 在渲染时插入图片。

> 上一篇文章说过，将 `_config.yml` 中的 `post_asset_folder` 设置为 `true` 时，每新建一篇文章就会在其同级文件夹新建一个同名文件夹，用于存放图片等附件。

其实可以发现，插入图片最重要的就是图片的路径。

「路径」有「绝对路径」和「相对路径」之分：

- 插入绝对路径是不可以的，因为 Hexo 渲染时会将 `source` 中的文件拷贝到 `public`，文件的绝对路径肯定会发生变化；

- 插入相对路径（图片相对于 Markdown 文档的路径）看似是可行的，但

  - 在 `source` 中，Markdown 文档在图片文件的上一层目录中；
  - 在 `public` 中，Markdown 渲染成的 html 文件和图片文件在同一层。

  即 Hexo 的渲染也会改变图片和 Markdown 文档的相对路径。

这时候，可以借助 `hexo-asset-image` 插件：

```zsh
npm install hexo-asset-image –-save
```

注意，安装好插件之后，一定要修改 `/node_modules/hexo-asset-image/index.js` 文件为：

```javascript
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
        var link = data.permalink;
    if(version.length > 0 && Number(version[0]) == 3)
       var beginPos = getPosition(link, '/', 1) + 1;
    else
       var beginPos = getPosition(link, '/', 3) + 1;
    // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
    var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];

      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
        if ($(this).attr('src')){
            // For windows style path, we replace '\' to '/'.
            var src = $(this).attr('src').replace('\\', '/');
            if(!/http[s]*.*|\/\/.*/.test(src) &&
               !/^\s*\//.test(src)) {
              // For "about" page, the first part of "src" can't be removed.
              // In addition, to support multi-level local directory.
              var linkArray = link.split('/').filter(function(elem){
                return elem != '';
              });
              var srcArray = src.split('/').filter(function(elem){
                return elem != '' && elem != '.';
              });
              if(srcArray.length > 1)
                srcArray.shift();
              src = srcArray.join('/');
              $(this).attr('src', config.root + link + src);
              console.info&&console.info("update link as:-->"+config.root + link + src);
            }
        }else{
            console.info&&console.info("no src attr, skipped...");
            console.info&&console.info($(this));
        }
      });
      data[key] = $.html();
    }
  }
});
```

否则会出现路径问题。

修改完成之后，无论是 Markdown 文档的本地预览还是 html 部署后，都可以完美地查看图片。

但还没有结束。

问题正是伴随着我的自定义界面而出现的： `hexo-asset-image` 插件会影响 Hexo 对 html 文件的渲染。

不仅如此，平时如果用 Markdown 写一些带有图片的报告，它的分享就会变得很麻烦，因为只能导出为 pdf 文档进行分享，才能让图片嵌入其中。

究其原因，就是 Markdown 文档和图片文件之间的联系太弱了，弱到移动文件的位置就会使它们之间的联系断开。

这时回到开始时的问题：Markdown 文档引用图片有绝对路径和相对路径之分，那么可不可以找到一个永久不变的「绝对路径」呢？

可以，那就是「图床」。

图床，就是把图片上传到云端服务器，然后通过插入链接来进行访问。它不仅可以解决图片文件路径的问题，还可以节省本地服务器空间并加快图片加载速度。

- 推荐的图床：[阿里云 OSS](https://www.aliyun.com/product/oss)
- 推荐的图床上传工具：[PicGo](https://github.com/Molunerfinn/PicGo)

上传到图床的图片文件名注意不要有 `+` 等符号，否则会上传失败。若上传图片失败，请检查图片文件名。

最新版 Typora 已经支持和 PicGo 梦幻联动，毫不夸张，可以让你写 Markdown 的效率翻倍。

最后别忘了，卸载 `hexo-asset-image` 插件。

> 参考
>
> - [Hexo 引用本地图片以及引用本地任意位置图片的一点思路 | 禾七博客](https://leay.net/2019/12/25/hexo/)
