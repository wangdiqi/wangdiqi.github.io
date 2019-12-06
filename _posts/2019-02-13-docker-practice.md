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




## k8s

kubectl get svc -n cloud | grep casr   //get port
kubectl get pod -n cloud -o wide | grep casr     //get pyhisical machine
ifconfig | egrep 10.   //get 
