---
layout: post
title: "测试框架"
subtitle: "测试框架"
date: 2018-11-03 09:50:00
author: "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 测试
---

~~~
               LoadRunner Jmeter  Locust  Tsung

授权方式       商业收费    开源免费   开源免费  开源免费

开发语言        C/Java     Java      Python    Erlang

测试脚本形式    C/Java     GUI       Python    xml

并发机制       进程/线程   线程      协程      “轻量级”进程

单机并发能力   低          低        高        高

分布式压力     支持       支持       支持      支持

资源监控       支持      不支持     不支持    支持

报告与分析     完善      简单图表   简单图表   丰富图表

支持二次开发   不支持    支持       支持      支持
~~~




Kubernetes + Locust


ref: https://cloud.google.com/solutions/distributed-load-testing-using-kubernetes



https://github.com/locustio/locust(python)



https://github.com/myzhan/boomer(golang)



Locust 是一个开源负载测试工具。使用 Python 代码定义用户行为，也可以仿真百万个用户。Locust 是非常简单易用，分布式，用户负载测试工具。Locust 主要为网站或者其他系统进行负载测试，能测试出一个系统可以并发处理多少用户。Locust 是完全基于时间的，因此单个机器支持几千个并发用户。相比其他许多事件驱动的应用，Locust 不使用回调，而是使用轻量级的处理方式 gevent。



特性



使用纯 Python 代码编写用户测试场景；不需要 UIs 或者 XML


分布式&可伸缩 - 支持成千上万的用户


基于 Web 的 UI


可以测试任意系统；虽然 Locust 是面向 Web 的，但是也可以测试其他任意的系统


部署时，首先部署load testing Master，然后部署10个load testing Worker。



部署Locust Master后，您可以使用外部传输规则的外部IP地址访问Web界面。部署Locust工作器后，您可以启动模拟并检查通过Locust Web界面聚合的统计信息。



要扩大模拟的用户数量，您需要增加蝗虫工作者窗格的数量。复制控制器按照Locust工作控制器的指定部署十个Locust工作器pod。为了增加复制控制器部署的pod数量，Kubernetes提供了在不重新部署pod的情况下更改控制器大小的功能。例如，您可以使用kubectl命令行工具更改工作线程管理器的数量。



     locust的架构是经典压测架构的master->slave，开源有golang实现的slave(https://github.com/myzhan/boomer)。locust本身很轻量，虽然官方只提供了对http的支持，但是官方给出了接口，可以根据实际需要二次开发实现对非web组件的压测，例如websocket、mqtt、redis等。              

    websocket example   https://gist.github.com/reallistic/dae87055210af82ea6aa4851a60fde11   

    mqtt example            https://github.com/eclipse/paho.mqtt.golang







example :



from locust import HttpLocust, TaskSet, task


class WebsiteTasks(TaskSet):


    def on_start(self):


        self.client.post("/login", {


            "username": "test",


            "password": "123456"


        })


    @task(2)


    def index(self):


        self.client.get("/")


    @task(1)


    def about(self):


        self.client.get("/about/")


class WebsiteUser(HttpLocust):


    task_set = WebsiteTasks


    host = "http://www.dui.ai"


    min_wait = 1000


    max_wait = 5000


先模拟用户登录系统，然后随机地访问首页（/）和关于页面（/about/），请求比例为2:1；并且，在测试过程中，两次请求的间隔时间为1~5秒间的随机值。







Locust 参数化 (测试数据)



这一项极其普遍，主要是用在测试数据方面。但通过归纳，发现其实也可以概括为3中类型。



循环取数据，数据可重复使用：e.g. 模拟3用户并发请求网页，总共有100个URL地址，每个虚拟用户都会依次循环加载这100个URL地址；


保证并发测试数据唯一性，不循环取数据：e.g. 模拟3用户并发注册账号，总共有90个账号，要求注册账号不重复，注册完毕后结束测试；


保证并发测试数据唯一性，循环取数据：模拟3用户并发登录账号，总共有90个账号，要求并发登录账号不相同，但数据可循环使用。
