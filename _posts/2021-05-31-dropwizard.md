---
layout: post
title: "dropwizard学习"
subtitle: "基础知识"
date: 2021-5-31 10:00:00
author: "diqi"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - java
---

## logback配置

~~~
logging:
  level: INFO
  appenders:
    - type: console
      logFormat: "[%d{ISO8601,UTC}] %c | [%F:%L] | %m%n%rEx"
      timeZone: UTC
      includeCallerData: true
~~~

includeCallerData: 在需要打印行号、函数名等需要堆栈信息的情况下需要设置为true  
%F: 打印方法名  
%L: 打印行号  




## TODO

~~~
final PolymorphicAuthDynamicFeature<Principal> feature = new PolymorphicAuthDynamicFeature<>(
                ImmutableMap.of());
final AbstractBinder binder = new PolymorphicAuthValueFactoryProvider.Binder<>(
                ImmutableSet.of());
environment.jersey().register(feature);
environment.jersey().register(binder);
~~~
