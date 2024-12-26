---
url: overleaf-citation-error
title: Overleaf LaTeX citation 无法正常显示问题解决
categories: [技术]
tags: [LaTeX, Overleaf]
mathjax: false
copyright: true
comments: true
date: 2024-12-26 22:24:42
link:
post_link:
---

神奇的 Overleaf。

<!--more-->

## 问题

今天在用 Overleaf writing 的时候，使用 `\cite{paper}` 出现 `?` 而非 ref 序号。`main.tex` 的具体内容如下：

```latex
\usepackage{filecontents}

\begin{filecontents}{\jobname.bib}

@article{you2023,
  author =       {You},
  title =        {Overleaf is bad},
  publisher =    {Science},
  year =         2023
}
@article{you2024,
  author =       {You},
  title =        {Overleaf is not what you need at all},
  publisher =    {Science},
  year =         2024
}

\end{filecontents}

\begin{document}

Overleaf is bad~\cite{you2023}.

Overleaf is not what you need at all~\cite{you2024}.

DO NOT use Overleaf~\cite{you2024,you2023}.

\bibliographystyle{plain}
\bibliography{\jobname}

\end{document}
```

显示效果为：

```
Overleaf is bad [?].
Overleaf is not what you need at all [?].
DO NOT use Overleaf [?, ?].
```

但十分奇怪的是，并不是开始时就出现此问题。当我新建项目并导入 `TeX` 文件时，Overleaf 可以正常编译并显示 ref 序号：

```
Overleaf is bad [1].
Overleaf is not what you need at all [2].
DO NOT use Overleaf [1, 2].
```

当删除 ref 内容再重新粘贴回去时，序号就会变为 `?`，且重新编译不能解决。

于是搜索了一下相关问题，但并没有找到类似问题的讨论。尝试了清除缓存、检查并修改文件名、检查 `sty` 文件、调整文件路径、更换编译器等操作后，均无法解决此问题。

## 解决方案

Overleaf 内置的编译器（`pdfLaTeX` 和 `LaTeX`）对 `\jobname` 的支持有问题，因此需要将隐式的 `\jobname` 替换为显式的文件名称。

以 `main.tex` 举例，需要将 `\jobname` 替换为 `main`：

```latex
\usepackage{filecontents}

\begin{filecontents}{main.bib}

@article{you2023,
  author =       {You},
  title =        {Overleaf is bad},
  publisher =    {Science},
  year =         2023
}
@article{you2024,
  author =       {You},
  title =        {Overleaf is not what you need at all},
  publisher =    {Science},
  year =         2024
}

\end{filecontents}

\begin{document}

Overleaf is bad~\cite{you2023}.

Overleaf is not what you need at all~\cite{you2024}.

DO NOT use Overleaf~\cite{you2024,you2023}.

\bibliographystyle{plain}
\bibliography{main}

\end{document}
```

但如果只是这样替换，并无法解决问题。还需要在同目录下新建 `main.bib` 文件，将 `\begin{filecontents}{main.bib}` 和 `\end{filecontents}` 之间的内容拷贝至 `main.bib`。此时可以删除 `main.tex` 中的对应内容。

此时，重新编译即可解决问题：

```
Overleaf is bad [1].
Overleaf is not what you need at all [2].
DO NOT use Overleaf [1, 2].
```
