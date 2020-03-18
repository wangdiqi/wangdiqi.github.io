---
layout: post
title: "文件同步服务"
subtitle: "文件同步"
date: 2018-11-03 09:50:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 杂
---

1.主动watch目录, 同步备份
使用rsync+inotify,  sersync  https://github.com/wsgzao/sersync 
* 多线程
* 过滤队列
* 支持失败重试

- 常作为文件备份使用
- 配置从文件读取
- 不支持回调 



2.httpserver + wget
http文件服务器监听目录， wget递归下载

3.文件同步工具 rsync server + rsync client
