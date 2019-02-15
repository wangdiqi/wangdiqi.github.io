---
layout: post
title: "profile工具"
subtitle: "profile"
date: 2019-02-14 15:29:00
author: "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - profile
---

> "Let's go"

## ps

## netstat(废弃)
~~~
netstat -np[a,u,p,l,e]
~~~

## ss(替代netstat)
一般无需root权限，若想列出所有连接状态，可能需要root  
查看socket信息，连接状态，端口，pid等  
~~~
ss -nt state establish dport = 8080 or sport = 8080
~~~

## top
一般无需root权限  
查看cpu、内存利用率  
~~~
top -d 1   //检查CPU，LOAD， 进程CPU占比，内存消耗，user cpu，sys cpu， idle cpu
~~~

## iostat
~~~
iostat -d -x 1 // 检查IO情况，吞吐量，IOPS
~~~

## vmstat
一般无需root权限  
查看running和block进程数量  
内存信息  
磁盘读写信息  
~~~
vmstat -1   //检查running 进程，block进程，cache 内存等等
~~~

## iftop
需要root权限  
可以查出哪条连接占用带宽，ifstat不具备
~~~
iftop -nNP   // 检查网络吞吐，带宽
~~~

## ifstat
按设备网卡查看流量
只能看带宽总和

## nethogs
按进程查看流量占用

## iptraf
按连接/端口查看流量


## ethtool
诊断工具

## dstat

## slurm

## nload

## bmon

## tcpdump
抓包工具

## strace 内核态

## ltrace 用户态

## /proc 目录

## 火焰图
分析cpu占比

CPU usage is high：CPU flamegraph with perf or with systemtap (like sample-bt)  

CPU is low, then you should sample an off-CPU flamegraph to analyze (using perf or using a systemtap tool like sample-bt-off-cpu)

#### perf

#### systemtap

#### dtrace


## 备注

~~~
用途              net-tool(被淘汰)     iproute2
地址和链路配置       ifconfig            ip addr, ip link
路由表              route               ip route
邻居                arp                 ip neigh
VLAN               vconfig             ip link
隧道                iptunnel            ip tunnel
组播                ipmaddr             ip maddr
统计                netstat             ss
~~~
