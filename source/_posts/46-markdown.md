---
url: markdown
title: 'Mark ⬇️，能力 ⬆️'
date: 2021-02-22 14:53:51
categories: [技术]
tags: [Markdown]
mathjax: true
---

看一看 Markdown 如何让「写文章」更加容易。

<!--more-->

你应该知道，通过 `hexo n new_post` 可以新建一篇文章。新建的文章会出现在 `/source/_posts/` 中，是一个扩展名为 `md` 的文档。

熟悉 Microsoft Word 的同学可能印象中文档的扩展名都是 `docx` 或 `doc`、`txt`、`rtf` 等，`md` 扩展名意味着这是一个新的格式，其实是一种新的语言——

<img src="https://i0.hdslb.com/bfs/album/b920a1e63dadc80ea0ea79e18324b68e960b8b15.png" alt="Markdown" style="zoom:50%;" />

**Markdown**。

# 0 什么是 Markdown？

Markdown 是一种轻量级的文本标记语言，它可以让纯文本很容易地变成你想要的样式。你不需要像 Microsoft Word 一样关心标题应该是多大字号，段前距应该设为多少，列表要如何编号，只要在**纯文本**前后增加一些标记符号（例如 `#` `-` `*` `>`），就能毫不费力地写出工整精美的文章。

人类的思考活动正是因为有了「笔」，才使得虚无变成了实体。毫不夸张地说，Markdown 是比特世界赠予写作者的「笔」。

# 1 为什么要用 Markdown？

## 上手容易

不要胆怯于 Markdown 陌生的使用方式，也不要不屑于它复杂的标记字符，实际上只要几分钟就可以 [上手 Markdown](https://commonmark.org/help/tutorial/)，然后你可能再也不想碰 Word 了。

## 书写流畅

富文本编辑器（例如 Word）通过点击图形化的功能按钮来实现排版，这时「输入文字」和「编辑文字」两个动作是不连续的。而 Markdown 则将这两个动作合并为一个「输入字符」的动作，通过标记字符去赋予文字不同格式。视线不离光标、双手不离键盘，**易读易写**（easy-to-read and easy-to-write），Markdown 让任何与文字打交道的人专注于写作，不用操心排版。

## 格式统一

如果你以 docx 格式来分享一份文档，你可能会发现同一份 Word 文档通过不同的设备、版本、软件打开可能显示不一样的效果。而 Markdown 可以 **Write once, export everywhere**。而且由于 md 格式的文件本质上仍是纯文本，所以也不会出现打不开的情况。不管在电脑上还是手机上，用 Markdown 写出来的文章都能带来舒适、统一、美好的阅读体验，而且支持导出为 PDF、Word、HTML、图片、甚至微信公众号等各种格式。

# 2 如何使用 Markdown？

## 2.0 入门

一开始你只要记住以下符号（英文半角），就能写出排版清爽的文章了。

| 标记符号                    | 效果     |
| --------------------------- | -------- |
| `#` + `空格` + `文本`       | 一级标题 |
| `##` + `空格` + `文本`      | 二级标题 |
| `###` + `空格` + `文本`     | 三级标题 |
| `-` + `空格` + `文本`       | 无序列表 |
| `1` + `.` + `空格` + `文本` | 有序列表 |
| `>` + `空格` + `文本`       | 引用     |

如果有些文字需要特殊说明：

| 标记符号               | 效果       |
| ---------------------- | ---------- |
| `**` + `加粗` + `**`   | **加粗**   |
| `*` + `斜体` + `*`     | *斜体*     |
| `~~` + `删除线` + `~~` | ~~删除线~~ |

如果想插入图片或超链接：

```
![描述](图片地址)

[描述](链接地址)
```

想动手试一试？下载一个Markdown 编辑器， Keep calm and Markdown。

- Windows 和 macOS 用户推荐 [Typora](https://typora.io/)。

  ![](https://i.loli.net/2019/07/15/5d2b601b5f46477978.gif)

- Android 用户推荐 [纯纯写作](https://writer.drakeet.com/)。

  ![](https://i.loli.net/2019/01/06/5c3199ab7f96e.jpg)

- iOS 和 iPadOS 用户推荐 [熊掌记](https://bear.app/cn/)。

  ![image-20210222121028076](https://i0.hdslb.com/bfs/album/baa275eb18231260d2490dff33a9c18661b0ee0f.png)

## 2.1 进阶

以上就是最基础的 Markdown 语法，你看到这里就可以开始动笔尝试了。如果你追求书写风格和条理性，建议继续看。

下文主要讲为了使源码具有良好的可读性、可移植性和可维护性，用 Markdown 写文章应该遵守哪些书写风格。你也不必一下子全都记住，需要实现哪个功能再来查阅或者问 Google 也不迟。

### 2.1.0 标题

- 第一个标题 `#` 应当是一个一级标题，并且应该尽可能和文件名称保持一致。第一个一级标题会被用作页面标题。第一个标题之下可以写上有关作者的信息。

- 如果文章很长，建议在一级标题下用 `[TOC]` 生成文章目录。

- 正文的标题应从 `##` 开始，`#` 要留给整篇文章的标题（也可以不写）。这样的大纲结构可以很方便地转换为思维导图（Markdown to [Xmind](https://www.xmind.cn/) / [MindNode](https://mindnode.com/) / [幕布](https://mubu.com/) / [百度脑图](http://naotu.baidu.com/)）。

  ```
  # 一级标题（h1）：文章的标题

  ## 二级标题（h2）：正文的大标题

  ### 三级标题（h3）：正文的小标题
  ```

- Markdown 最大支持 `######` 六级标题，一般文章建议使用到 `###` 三级标题为止，更低级的标题建议使用列表代替。

  ```
  一般文章的标题层级

  ### 三级标题（h3）：正文的小标题

  **四级标题 A**
  - 论据 1（五级标题）
    - 细分论据 1（六级标题）
    - 细分论据 2（六级标题）
    - 细分论据 3（六级标题）
  - 论据 2
  - 论据 3

  **四级标题 B**
  1. 论据 1
  2. 论据 2
  3. 论据 3

  **四级标题 C**
  - 论据 1
  - 论据 2
  - 论据 3
  ```

- 各级标题应当连续，如在二级标题下不能直接出现四级标题。

- 如果同级标题只有一个，那么考虑不对它孤立编号。

- 标题要简短，`#` 后要加空格，结尾不应带标点符号。

- 在 Markdown 源码中，标题前后应各空一行，相当于段前距和段后距。

- 大标题和小标题之间要有内容过渡，用于引出或概括下文。

  ```
  # Title

  开门见山地告诉读者这篇文章讲什么？

  ## What

  问题是什么？

  ## Why

  为什么会出现这样的问题？

  ## How

  该怎么解决问题？

  ## See also

  在底部为想了解更多相关知识的读者放置链接。

  - https://link-to-more-info
  ```

  可以发现，本文就是按照这样的逻辑展开的。

- 使用分隔线 `---`、`***` 或 `+++`，可以将上下两部分分隔。

### 2.1.1 列表

Markdown 列表有「无序列表」和「有序列表」，二者还可以结合成「嵌套列表」。更高级的，还有「任务列表」。

#### 2.1.1.0 无序列表

使用 `‐`、`*` 或 `+` 并跟随一个空格来表示无序列表，建议使用 `‐`。

**建议**

```
- 我是谁
- 我从哪里来
- 我到哪里去
```

**不建议**

```
* 我是谁
* 我从哪里来
* 我到哪里去

---

+ 我是谁
+ 我从哪里来
+ 我到哪里去
```

**为什么**

- 星号 `*` 可能与加粗和斜体符号产生混淆；
- 通过键盘输入加号 `+` 比输入减号 `-` 麻烦。

**与其他语法结合使用**

```
- **《春日》**：等闲识得东风面，万紫千红总是春。
- **《春草》**：萋萋总是无情物，吹绿东风又一年。
- **《墨梅》**：犹恨东风无意思，更吹烟雨暗黄昏。
```

**预览**

- **《春日》**：等闲识得东风面，万紫千红总是春。
- **《春草》**：萋萋总是无情物，吹绿东风又一年。
- **《墨梅》**：犹恨东风无意思，更吹烟雨暗黄昏。

#### 2.1.1.1 有序列表

有序列表的”序“，可以通过手动和自动来控制。

##### 2.1.1.1.0 手动排序

```
1. 斜月沉沉藏海雾，碣石潇湘无限路。
2. 不知乘月几人归，落月摇情满江树。
3. 春江潮水连海平，海上明月共潮生。
```

**预览**

1. 斜月沉沉藏海雾，碣石潇湘无限路。
2. 不知乘月几人归，落月摇情满江树。
3. 春江潮水连海平，海上明月共潮生。

对于比较短的、很少修改的有序列表，请按顺序标号，这样源码读起来更加容易。

##### 2.1.1.1.1 自动排序

对于比较长的、可能会修改的列表（尤其是很长的嵌套列表），请使用「懒人编号法」。即使有新的列表项“插队”，把序号弄乱了也没关系，Markdown 编辑器自动会对序号进行纠错。

```
1.  Foo.
1.  Bar.
    1. Foofoo.
    1. Barbar.
1.  Baz.
```

**预览**

1. Foo.
2. Bar.
   1. Foofoo.
   2. Barbar.
3. Baz.

#### 2.1.1.2 嵌套列表

通过缩进四个空格或一个 Tab，可以嵌套列表。

```
1.  不知乘月几人归，落月摇情满江树。
	- 与君吟弄风月，端不负平生。
	- 对秋深，离恨苦，数夜满庭风雨。
	- 五月畲田收火米，三更津吏报潮鸡。
2.  人姝丽，粉香吹下，夜寒风细。
	- 弓弦抱汉月，马足践胡尘。
	- 寒月悲笳，万里西风瀚海沙。
	- 东堂坐见山，云风相吹嘘。
3.  沅溪夏晚足凉风，春酒相携就竹丛。
	- 白发渔樵江渚上，惯看秋月春风。
	- 归来独卧逍遥夜，梦里相逢酩酊天。
	- 致君尧舜上，再使风俗淳。
```

**预览**

1. 不知乘月几人归，落月摇情满江树。
   - 与君吟弄风月，端不负平生。
   - 对秋深，离恨苦，数夜满庭风雨。
   - 五月畲田收火米，三更津吏报潮鸡。
2. 人姝丽，粉香吹下，夜寒风细。
   - 弓弦抱汉月，马足践胡尘。
   - 寒月悲笳，万里西风瀚海沙。
   - 东堂坐见山，云风相吹嘘。
3. 沅溪夏晚足凉风，春酒相携就竹丛。
   - 白发渔樵江渚上，惯看秋月春风。
   - 归来独卧逍遥夜，梦里相逢酩酊天。
   - 致君尧舜上，再使风俗淳。

**Tips**

- `Tab` 降低一级（一级缩进）
- `Shift + Tab` 提升一级（取消一级缩进）

#### 2.1.1.3 任务列表

```
- [ ] 锻炼
- [ ] 学习
- [x] 吃饭
- [x] 睡觉
```

**预览**

- [ ] 锻炼
- [ ] 学习
- [x] 吃饭
- [x] 睡觉

### 2.1.2 引用

在每一行使用 `>` 符号后接一个空格，包括换行的句子。

**举例**

```
> 我们是为人民服务的，所以，我们如果有缺点，就不怕别人批评指出。
> ——毛泽东：《为人民服务》（1944 年 9 月 8 日）
```

**预览**

> 我们是为人民服务的，所以，我们如果有缺点，就不怕别人批评指出。
> ——毛泽东：《为人民服务》（1944 年 9 月 8 日）

### 2.1.3 表格

#### 2.1.3.0 普通表格

```
|车次|出发站|出发时间|到达站|到达时间|
|-----|---|-----|-------|-------|
|G3654|赤峰|14:25|北京朝阳|17:07|
```

**预览**

| 车次  | 出发站 | 出发时间 | 到达站   | 到达时间 |
| ----- | ------ | -------- | -------- | -------- |
| G3654 | 赤峰   | 14:25    | 北京朝阳 | 17:07    |

#### 2.1.3.1 HTML 表格

```html
<table>
	<tr>
  	<td>车次</td>
  	<td>出发站</td>
  	<td>出发时间</td>
    <td>到达站</td>
  	<td>到达时间</td>
  </tr>
  <tr>
  	<td>G3654</td>
    <td>赤峰</td>
  	<td>14:25</td>
    <td>北京朝阳</td>
  	<td>17:07</td>
  </tr>
</table>
```

**预览**

<table>
	<tr>
  	<td>车次</td>
  	<td>出发站</td>
  	<td>出发时间</td>
    <td>到达站</td>
  	<td>到达时间</td>
  </tr>
  <tr>
  	<td>G3654</td>
    <td>赤峰</td>
  	<td>14:25</td>
    <td>北京朝阳</td>
  	<td>17:07</td>
  </tr>
</table>

#### 2.1.3.2 设置对齐方式

```
| 居中对齐 | 右对齐 | 左对齐 |
|:-------:|------:|:------|
| 居中对齐 | 右对齐 | 左对齐 |
```

**预览**

| 居中对齐 | 右对齐 | 左对齐 |
| :------: | -----: | :----- |
| 居中对齐 | 右对齐 | 左对齐 |

#### 2.1.3.3 Tips

- Markdown 是轻量级的标记语言，所以不支持合并和拆分单元格。对于复杂表格，可以使用 HTML 的 `<table>` 标签标记。
- 在 Markdown 中的表格应尽可能小，大型表格建议改用列表。
- 单元格内输入 `<br />` 可以换行（但不建议使用）。
- 把 Excel 表格复制粘贴到某些 Markdown 编辑器（例如 Typora）可以直接转换为 Markdown 形式的表格。

### 2.1.4 链接

**语法**

```
[描述](链接地址)
```

**举例**

```
[Super Blog](https://superpung.com)
```

**预览**

[Super Blog](https://superpung.com/)

### 2.1.5 图片

**语法**

```
![描述](图片地址)
```

**举例**

```
![Exitalk](https://i0.hdslb.com/bfs/album/d9137e1e399c83687fddf34c9aa2cef485d03afd.png)
```

**预览**

![Exitalk](https://i0.hdslb.com/bfs/album/d9137e1e399c83687fddf34c9aa2cef485d03afd.png)

**图床**

关于为什么使用图床，请参见我的 [另一篇文章](https://superpung.com/blog-insert-images/)。

**本地图片**

Typora 支持插入本地图片，但是更改图片的路径和名称时，图片就失效了。如果文章已经完稿了，可以把 Markdown 导出为 PDF 文档，这样图片就嵌入进去了。

### 2.1.6 代码

Markdown 支持「行内代码」和「代码块」，使用「反引号」，在键盘的 Tab 键上方（英文模式）。

#### 2.1.6.0 行内代码

**语法**

```
`Markdown` 是一种轻量级标记语言。
```

**预览**

`Markdown` 是一种轻量级标记语言。

**Tips**

反引号前后各空一格。

#### 2.1.6.1 代码块

**语法**

```
```python
print "Hello, world!"
```

**预览**

```python
print "Hello, world!"
```

**diff 代码对比**

```
```diff
function addTwoNumbers(num1, num2) {
-  return 1 + 2
+  return num1 + num2
}
```

**预览**

```diff
function addTwoNumbers(num1, num2) {
-  return 1 + 2
+  return num1 + num2
}
```

### 2.1.7 换行

{% note info %}

以下键盘快捷键（语法）可能仅支持 Typora。

{% endnote %}

`Enter` = 换行 + 空行 = (`Shift` + `Enter`) × 2。

即按下回车键创建一个新段落（段与段之间加入空行）。

`Shift` + `Enter` = 换行（但是不会产生空行）。

**举例**

```
`enticing` [ɪn'taɪsɪŋ] （事物）诱人的，有吸引力的；迷人的 ⏎
- Her neck was short but rounded and her arms plump and enticing. ⇧⏎
  她的脖子短，但浑圆可爱；两臂丰腴，也很动人。——《飘》 ⏎

- This was enticing to Wozniak, even more than any prospect of getting rich. ⇧⏎
  这句话对沃兹尼亚克的诱惑太大了，比变成富人的诱惑还要大。——《乔布斯传》 ⏎
```

**预览**

`enticing` [ɪn'taɪsɪŋ] （事物）诱人的，有吸引力的；迷人的

- Her neck was short but rounded and her arms plump and enticing.
  她的脖子短，但浑圆可爱；两臂丰腴，也很动人。——《飘》
- This was enticing to Wozniak, even more than any prospect of getting rich.
  这句话对沃兹尼亚克的诱惑太大了，比变成富人的诱惑还要大。——《乔布斯传》

### 2.1.8 公式

使用 MathJax 引擎，可以方便地用 $\LaTeX$ 方法在 Markdown 中插入数学公式。

**行内公式**

```latex
这是一个行内公式$\frac{n!}{k!(n-k)!} = \binom{n}{k} \quad \mbox{for }\ 0\leq k\leq n$
```

**预览**

这是一个行内公式$\frac{n!}{k!(n-k)!} = \binom{n}{k} \quad \mbox{for }\ 0\leq k\leq n$

**块级公式**

```latex
这是一个块级公式$$\frac{n!}{k!(n-k)!} = \binom{n}{k} \quad \mbox{for }\ 0\leq k\leq n$$
```

**预览**

这是一个块级公式$$\frac{n!}{k!(n-k)!} = \binom{n}{k} \quad \mbox{for }\ 0\leq k\leq n$$

### 2.1.9 其他

关于 Markdown 还有许多其他语法，如时序图、流程图等。由于不常用，本文就不再过多介绍了。

# 3 什么时候能用 Markdown？

了解了如何使用 Markdown，下面就来说说什么时候能用它。

## 3.0 Markdown 的局限性

先说说什么时候不能用 Markdown。通过前文的介绍，再加上和 Word 的对比，可以发现它仍然有局限性：

### 对于「段落」

**Markdown 无法对段落进行灵活处理**。在 Word 中你可以随意插入文本框并调整它的位置，可以设置段落的居中对齐或右对齐。相比 Markdown 只能线性的对文字排版，专门的排版软件无疑更能满足专业的需求。

### 对于「图片」

**Markdown 对图片的排版能力很差**。像 Word 一样，现在很多编辑器都支持了图文混排。但受制于纯文本格式，Markdown 编辑器几乎不可能做到 Word 一样对图片**灵活**地调整位置和大小，更不用说文字围绕图片进行自适应排版之类的效果。

### 对于「表格」

**Markdown 对表格的排版能力很差。**Markdown 不支持合并和拆分单元格。如果想在 md 文档中插入复杂表格，只能用 HTML 实现。

### 总结

所以，Markdown 并不适合对排版格式要求较高的文档进行排版。

## 3.1 Markdown 的广泛应用

即使 Markdown 有很大的局限性，但仍然不能否定它的强大。

感谢开发者们源源不断的创意，为我们提供了极为丰富的工具选择。工具的多样，让 Markdown 能渗透进更多样的场景。Write once, export everywhere：写博客、写邮件、排推文、做 PPT 等。

### 公众号排版

- [Markdown Nice](https://mdnice.com/)
- [微信 Markdown 编辑器](https://doocs.github.io/md/)
- [可能吧公众号 Style 一键转换器](https://knb.im/mp/)

### 简历排版

- [冷熊简历](http://cv.ftqq.com/#)
- [resume.mdnice.com](https://resume.mdnice.com/)
- [Resumd](https://resumd.t9t.io/)

### 邮件排版

[Markdown Here](https://markdown-here.com/) 是一个浏览器扩展插件，可以将浏览器中编辑器（例如 email 正文）里的 Markdown 文本转换成渲染过后的 HTML，并且支持自定义 CSS。

### 转换为 Word

安装 Pandoc 后，[Typora](https://typora.io/) 可以导出 Word。

### 转换为 Mind Map

- 对于 [Xmind](https://www.xmind.cn/) / [MindNode](https://mindnode.com/) / [百度脑图](http://naotu.baidu.com/) 等思维导图工具，直接导入 md 文档。
- 对于幕布，可以用 [Typora](https://typora.io/) 导出 opml 格式，再导入。
- [Markmap](https://markmap.js.org/)：`Mark`down + Mind`map`，使用思维导图可视化 Markdown。

### 从网页导出 md 格式

借助 Chrome 浏览器插件 [简悦 - SimpRead](https://chrome.google.com/webstore/detail/simpread-reader-view/ijllcpnolfcooahcekpamkbidhejabll) 可以导出 md 格式的网页。

### 导出 Markdown 源代码图片

使用 [Carbon](http://carbon.now.sh/)，语言选择 `Markdown`，书写或者粘贴 Markdown 源代码，就可以导出 PNG 像素图或者 SVG 矢量图。

---

> 参考
>
> - [Markdown 入门教程及书写风格指南 | 庭说](https://tingtalk.me/markdown/)
> - [Markdown 完全入门（上）](https://sspai.com/post/36610)
> - [Markdown 完全入门（下）](https://sspai.com/post/36682)
> - [Markdown Syntax - Daring Fireball](https://daringfireball.net/projects/markdown/syntax)
> - [Google Markdown 书写风格指南](https://www.jianshu.com/p/3beac9fd6496)
> - [Markdown Style Guide by Google](https://github.com/google/styleguide/blob/gh-pages/docguide/style.md)
