---
layout: post
title: "freeswitch"
subtitle: "VoIP"
date: 2020-04-17 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - freeswitch
---

> "Let's go"

## 安装

库
libsqlite3-dev
libcurl4-openssl-dev
libspeexdsp-dev
libldns-dev
libedit-dev
libavformat-dev


libav项目


命令
yasm


## 基本概念

### 工作流程

Channel状态机

~~~
NEW -->INIT-->ROUTING-->EXECUTE-->HANGUP-->REPORTING-->DESTROY
               ^              |
               |<--Transfer<--
~~~

当新建(NEW)一个Channel时，会进行初始化(INIT)，进入路由(ROUTING)阶段，也就是查找解析Dialplan阶段；  
找到合适路由后执行(EXECUTE)一系列动作，最后无论哪一方挂机，都会进入挂机(HANGUP)阶段。后面的  
报告(REPORTING)阶段一般用于进行统计、计费等，最后将Channel销毁(DESTROY)，释放系统资源。



### 常用命令

~~~
呼叫1000这个用户后，便执行echo这个程序
originate user/1000 &echo
~~~

~~~
列出某个profile的状态
sofia status profile internal

查看注册用户
sofia status profile internal reg

过滤某些符合条件的用户
sofia status profile internal reg 1000

列出某个特定用户
sofia status profile internal user 1000

列出网关状态
sofia status gateway gw
~~~


