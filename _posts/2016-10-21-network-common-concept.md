---
layout: post
title: "计算机网络基本知识"
subtitle: "一些基本概念"
date: 2016-10-21 21:00:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - network
---

> "基本概念"

### IP地址分类

|类别&nbsp;&nbsp;&nbsp;|最大网络数&nbsp;&nbsp;&nbsp;|IP地址范围&nbsp;&nbsp;&nbsp;|最大主机数&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;私有IP地址范围&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;子网掩码
|----------------------|:---------------|:---------------------------|-------:|:-----------
|A&nbsp;&nbsp;&nbsp;|126(2^7-2)|0.0.0.0-127.255.255.255|16777214(2^24 - 2)|&nbsp;&nbsp;&nbsp;10.0.0.0-10.255.255.255|&nbsp;&nbsp;&nbsp;255.0.0.0
|B&nbsp;&nbsp;&nbsp;|16384(2^14)|128.0.0.0-191.255.255.255 |65534(2^16 - 2)|&nbsp;&nbsp;&nbsp;172.16.0.0-172.31.255.255|&nbsp;&nbsp;&nbsp;255.255.0.0
|C&nbsp;&nbsp;&nbsp;|2097152(2^21)|192.0.0.0-223.255.255.255 |254(2^8 - 2)|&nbsp;&nbsp;&nbsp;192.168.0.0-192.168.255.255|&nbsp;&nbsp;&nbsp;255.255.255.0
