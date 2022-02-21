---
layout: post
title: "CS Architecture"
subtitle: "architecture"
date: 2021-12-15 09:50:00
author: "wangdiqi"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - architecture
---

> "Let's go"

## 常用的一些耗时知识

### 内存和L1、L2缓存

访问L2 cache 大概是 L1 cache的10几倍左右  
访问一次内存是访问L2 cache的20倍，是访问L1 cache耗时的200倍  
编写对cache友好的程序是至关重要的  

### 分支预测

因为现代CPU内部采用流水线方式来处理机器指令，因此在if指令未执行完时，后续指令已经被加到了流水线中，此时CPU需要猜测if语句是否为真，如果CPU猜对了，流水线继续运行，否则，流水线中已被执行的一部分指令就要作废。  
现代CPU分支预测成功率很高，大多数情况无需担忧分支预测错误造成的性能损失(5ns)  

### 内存、SSD、磁盘

读取1MB数据：  
磁盘花费的时间是SSD的20倍，是内存的80倍
SSD耗时是内存的4倍

### 网络和磁盘

网络IO不一定比磁盘IO慢。跟网络两端距离也有关系

### 网络

