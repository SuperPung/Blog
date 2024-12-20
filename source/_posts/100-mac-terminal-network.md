---
url: mac-terminal-network
title: Mac 终端无法联网问题解决
categories: [技术]
tags: [Mac, Apple]
mathjax: false
copyright: true
comments: true
date: 2024-12-21 02:04:16
link:
post_link:
---

神奇的 macOS。

<!--more-->

## 问题

今天本想装个东西，但 `brew update` 时一直报 `could not resolve host`。第一时间觉得是 Clash 的问题，于是设置了 proxy，但仍未解决，尝试 unset 但也无济于事。

后来发现不仅对于 `github.com` 报错，甚至对镜像 `mirrors.tuna.tsinghua.edu.cn` 也报错，`ping` 了一下 `github.com` 和 `baidu.com`，果然都不通。因此并不是 proxy 的问题。

尝试 `ping 8.8.8.8`，发现可以通。因此原因基本可以锁定是 DNS 设置的问题。

但之前从未更改过 Mac 的 DNS 设置，为什么终端突然不能联网了呢？尝试在 Settings - Wi-Fi - Details - DNS - DNS Servers 添加了两个阿里云的 DNS Server：`223.5.5.5`、`223.6.6.6`，并刷新 DNS 缓存：

```sh
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

稍等几分钟后，终端网络恢复正常。

## 总结

当 Mac 终端无法联网时，大多是因为两种情况：proxy 和 DNS，首先可以通过 `ping` 一个 IP（如 `ping 8.8.8.8`）检查是否是 DNS 的问题。

如果是 DNS 的问题，手动在 Settings - Wi-Fi - Details - DNS - DNS Servers 处添加公共 DNS Server（如 `223.5.5.5` 和 `223.6.6.6`）并刷新缓存：

```sh
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

这种方式在大多数情况下应该都可以解决 DNS 问题导致的网络连接问题。
