---
layout: post
title: "服务性能指标"
subtitle: "server performance"
date: 2019-05-24 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - performance
---

> "Let's go"


## QPS

第一种计算方式：
throughput (QPS) = processing power (CPU) / response time

第二种计算方式
QPS (TPS) = the number of concurrent / average response time or the number of concurrent = QPS* average response time



## 内存使用率计算公式:

(mem_total - (mem_free + mem_buffer + mem_cache)) / mem_total

例子：  
cat /proc/meminfo MemTotal: 8011936 kBMemFree: 227336 kBBuffers: 277872 kBCached: 1451828 kB  
计算方法：(8011936 - （227336 + 277872 + 1451828）) / 8011936  
计算结果：内存使用率约等于 75%。  


## 磁盘使用率公式:

max{vda1, vdb1}

docker容器挂载的磁盘存在快速销毁、短时间变动较大等问题暂不做统计，这边只统计最常用的 vda1或vdb1的最大值
