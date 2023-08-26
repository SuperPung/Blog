---
url: tju-drive-crawling
title: 简单的爬虫尝试
date: 2021-05-22 19:30:07
categories: [技术]
tags: [Python]
---

算是第一次写的一个小工具吧。

<!--more-->

GitHub：https://github.com/SuperPung/TJU-Drive-Crawling/

# 前言

回想起上一次写 Python 小项目，还是一年之前学习 Python 的时候。当时看书（pcc2e）时敲了一遍书后面几章的项目，尤其是外星人入侵的小游戏，很有趣。

时隔一年，面对每次打开都需要输入密码的天大云盘，还有好话老师留作业、批作业 follow his heart 的频率，我决定尝试写个小爬虫，能在云盘更新的时候提醒我。

爬虫，学 Python 之前就听说了，很喜欢这种懒人工具。前段时间开始经常逛 GitHub，也看到过一些小项目。于是我打开了许久未用过的 Pycharm，新建了一个项目。

> JetBrains 竟然不支持 edu 邮箱了，👴学生认证马上就到期，只能用学信网资料认证，好几天也不出结果……

写这篇文章时，这个小工具已经写完了。所以有些重构的过程可能会忘记了。

# Crawling

用 Chrome 抓包，分析到天大云盘的 url 实际上是 http://pan.tju.edu.cn:9123/v1/link?method=listdir，写一个请求头 `payload_header`：

```python
payload_header = {
    "Content-Type": "text/plain;charset=UTF-8",
    "Referer": "http://pan.tju.edu.cn/",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/90.0.4430.212 Safari/537.36 "
}
```

请求数据需要发出一个 json `data`：

```python
    data = {
        "link": link,
        "password": password,
        "by": "name",
        "sort": "asc"
    }
```

其中 link 就是天大云盘链接的后 32 位，password 就是密码。

最后通过 POST 请求得到响应：

```python
    s = requests.session()
    get_root = s.post(url, data=json.dumps(data), headers=payload_header)
    return get_root.text
```

# 响应分析

得到响应后，需要对内容分析。尝试用面向对象，但最终选择了函数。通过递归将文件目录输出。

每次分析后输出文件列表，以 json 格式存放到本地目录，下一次分析则与本地比较，这样就可以知道哪些文件更新了。

因为使用了递归，所以很多小功能又用函数重构了一下。

# 邮件服务

通过 smtplib 实现。

# 部署

部署到服务器也走了弯路，根本原因是路径的问题。

最后在脚本上先加一句 cd 解决。

# 后记

一共花了一天多一点的时间完成，还是很有趣的。
