---
layout: post
title: "docker学习"
subtitle: "docker"
date: 2019-02-13 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - docker
---

> "Let's go"

## 概述
docker 容器 =cgroup+namespace+secomp+capability+selinux

cgexec 是 cgroup 提供的一个工具，可以在启动时就将程序运行到某个 cgroup 中

nsenter 是一个 namespace 相关的工具，通过它可以进入某个进程所在的 namespace


## namespace

隔离作用，所有的namespace：cgroup/ipc/network/mount/pid/time/user/uts

~~~
man namespaces

具体看某个进程的namespace可以看
/proc/33090/ns/ 下的文件
~~~

## cgroups

~~~
man cgroups

具体看cgroup的配置
/sys/fs/cgroup/memory/docker/
~~~
