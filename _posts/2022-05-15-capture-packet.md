---
layout: post
title: "抓包工具"
subtitle: "network"
date: 2022-05-15 09:50:00
author: "wangdiqi"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - network
---

> "Let's go"

# wireshark

## 捕获过滤器

用于减少抓取的报文体积
使用BPF语法，功能相对有限

```
点击 捕获(capture)-> 选项(Option)

网卡设备/流量/捕获过滤器

输出 指定缓存文件 / 选项(显示、名称解析、自动停止抓包条件) 面板
```

### BPF 语法

1. primitives原语：由名称或数字，以及描述它的多个限定词组成

* qualifiers 限定词
** Type：设置数字或者名称所指示类型，例如 host www.baidu.com
1) host、port
2) net 设定子网：net 192.168.0.0 mask 255.255.255.0 等价于 net 192.168.0.0/24
3) portrange 设置端口范围：portrange 6000-8000
** Dir: 设置网络初入方向，例如 dst port 80
** Proto: 指定协议类型，例如 udp
** 其他

2. 原语运算符

* 与：&& 或者 and
* 或：|| 或者 or
* 非：! 或者 not

例如 src or dst portrange 6000-8000 && tcp or ip6

## 显示过滤器

对已经抓取到的报文过滤显示
功能强大

工具栏

## 导出报文

```
标记报文 Ctrl + M (macos 使用 Alt + M)

文件导出 选择特定的分组后选择Marked packets only
```

## 合并报文

```
文件 选择 merge
```
