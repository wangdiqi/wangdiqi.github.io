---
layout: post
title: "profile工具"
subtitle: "profile"
date: 2019-02-14 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - profile
---

> "Let's go"

## 常用的命令

#### ps
1. 一般无需root权限
2. 查看进程信息

#### top
1. 一般无需root权限  
2. 查看cpu利用率  
3. 查看内存利用率  
~~~
top -d 1   //检查CPU，LOAD， 进程CPU占比，内存消耗，user cpu，sys cpu， idle cpu
~~~

#### atop(功能很强大，神器)
1. 一般无需root权限
2. 可查看历史信息
3. 查看进程退出原因(需实现安装好atop后才会采集数据)
4. 查看进程
5. 查看磁盘
6. 查看网络
7. 查看内存
8. 查看交换空间


#### pidstat
1. 一般无需root权限
2. 查看cpu利用率
3. 查看io利用率
4. 查看实时优先级和调度策略信息
5. 查看内存和页错误信息
6. 查看栈使用信息
7. 查看所选择任务的线程信息
8. 查看任务上下文切换信息

#### dstat
1. 查看系统资源利用情况
2. 只有总的统计结果

#### sar
1. 查看系统资源利用情况

## 网络

#### netstat(废弃)
~~~
netstat -np[a,u,p,l,e]
~~~

#### ss(替代netstat)
1. 一般无需root权限，若想列出所有连接状态，可能需要root  
2. 查看socket信息，连接状态，端口，pid等  
~~~
ss -nt state establish dport = 8080 or sport = 8080
~~~

#### iftop
1. 需要root权限  
2. 查看设备带宽信息  
3. 可查看具体连接的带宽使用量，ifstat不具备  
~~~
iftop -nNP   // 检查网络吞吐，带宽
~~~

#### ifstat
1. 一般无需root权限  
2. 按设备网卡查看流量  
3. 只能看带宽总和  

#### nethogs
1. 需要root权限  
2. 列出所有进程的流量占用,  

#### iptraf
1. 需要root权限  
2. 按连接/端口查看流量  
3. 可以列出具体包的个数和大小  
4. 功能强大  

#### ethtool
1. 一般无需root权限
2. 查看和配置网卡设备
3. 功能很强大

## 磁盘IO

#### blktrace blkparse btt
1. blktrace需要root权限，采集数据
2. blkparse解析blktrace采集到的数据,可dump成bin文件
3. btt解析blkparse生成的bin文件

#### iostat
1. 一般无需root权限
2. 查看cpu利用率
3. 查看设备io利用率

await是队列等待时间(wait time)+磁盘工作时间(service time)，无法得到具体的磁盘工作时间
svctm应当忽略，官方已说明废弃
~~~
iostat -d -x 1 // 检查IO情况，吞吐量，IOPS
~~~

#### iotop
1. 需要root权限
2. 查看进程的io使用率

## 内存

#### vmstat
1. 一般无需root权限  
2. 查看running和block进程数量  
3. 内存信息  
4. 磁盘读写信息  
~~~
vmstat 1   //检查running 进程，block进程，cache 内存等等
~~~

#### pmap
pmap -d pid

## 程序性能分析

#### strace 内核态
跟踪程序的系统调用

#### ltrace
跟踪程序的库函数调用

#### pstack
跟踪程序进程栈

## /proc 目录

#### perf
可以追踪硬件

#### systemtap


#### dtrace

#### ftrace
最基本的功能是提供了动态和静态探测点，用于探测内核中指定位置上的相关信息。  
静态探测点，是在内核代码中调用ftrace提供的相应接口实现，称之为静态是因为，是在内核代码中写死的，静态编译到内核代码中的，在内核编译后，就不能再动态修改。在开启ftrace相关的内核配置选项后，内核中已经在一些关键的地方设置了静态探测点，需要使用时，即可查看到相应的信息。  
动态探测点，基本原理为：利用mcount机制，在内核编译时，在每个函数入口保留数个字节，然后在使用ftrace时，将保留的字节替换为需要的指令，比如跳转到需要的执行探测操作的代码。  

1. 只能在函数入口实现探测，systemtap可在函数中的任意位置实现探测
2. 实现函数替换后，原有函数的执行流程被替换成新函数，新函数执行完成后可以不再返回原函数流程执行。systemtap则会自动跳转回原有流程执行

#### 火焰图
分析cpu占比

CPU usage is high：CPU flamegraph with perf or with systemtap (like sample-bt)

CPU is low, then you should sample an off-CPU flamegraph to analyze (using perf or using a systemtap tool like sample-bt-off-cpu)


## 辅助命令

#### datamash
1. 计算均值和方差


## linux memory type
   Linux Memory Types
       For  our  purposes  there  are  three types of memory, and one is optional.  First is
       physical memory, a limited resource where code and data must reside when executed  or
       referenced.   Next  is  the  optional swap file, where modified (dirty) memory can be
       saved and later retrieved if too many demands are made on physical memory.  Lastly we
       have virtual memory, a nearly unlimited resource serving the following goals:

          1. abstraction, free from physical memory addresses/limits
          2. isolation, every process in a separate address space
          3. sharing, a single mapping can serve multiple needs
          4. flexibility, assign a virtual address to a file

       Regardless  of  which of these forms memory may take, all are managed as pages (typi‐
       cally 4096 bytes) but expressed by default in top as KiB (kibibyte).  The memory dis‐
       cussed  under  topic  `2c. MEMORY Usage' deals with physical memory and the swap file
       for the system as a whole.  The memory reviewed in topic `3. FIELDS  /  Columns  Dis‐
       play' embraces all three memory types, but for individual processes.

       For  each such process, every memory page is restricted to a single quadrant from the
       table below.  Both physical memory and virtual memory can include any  of  the  four,
       while  the  swap  file  only includes #1 through #3.  The memory in quadrant #4, when
       modified, acts as its own dedicated swap file.

                                     Private | Shared
                                 1           |          2
            Anonymous  . stack               |
                       . malloc()            |
                       . brk()/sbrk()        | . POSIX shm*
                       . mmap(PRIVATE, ANON) | . mmap(SHARED, ANON)
                      -----------------------+----------------------
                       . mmap(PRIVATE, fd)   | . mmap(SHARED, fd)
          File-backed  . pgms/shared libs    |
                                 3           |          4

       The following may help in interpreting process level memory values displayed as scal‐
       able columns and discussed under topic `3a. DESCRIPTIONS of Fields'.

          %MEM - simply RES divided by total physical memory
          CODE - the `pgms' portion of quadrant 3
          DATA - the entire quadrant 1 portion of VIRT plus all
                 explicit mmap file-backed pages of quadrant 3
          RES  - anything occupying physical memory which, beginning with
                 Linux-4.5, is the sum of the following three fields:
                 RSan - quadrant 1 pages, which include any
                        former quadrant 3 pages if modified
                 RSfd - quadrant 3 and quadrant 4 pages
                 RSsh - quadrant 2 pages
          RSlk - subset of RES which cannot be swapped out (any quadrant)
          SHR  - subset of RES (excludes 1, includes all 2 & 4, some 3)
          SWAP - potentially any quadrant except 4
          USED - simply the sum of RES and SWAP
          VIRT - everything in-use and/or reserved (all quadrants)

       Note:  Even  though  program  images and shared libraries are considered private to a
       process, they will be accounted for as shared (SHR) by the kernel.


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
