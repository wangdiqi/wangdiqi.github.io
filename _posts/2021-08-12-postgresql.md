---
layout: post
title: "postgresql 学习"
subtitle: "postgresql 学习"
date: 2021-08-12 09:50:00
author: "wangdiqi"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - database
---

> "Let's go"

# 常用命令

~~~
PGPASSWORD=pass1234 psql -U $username -d $database
\x 打开关闭多行展示
\dt 显示当前数据库表数据
\d $table_name 显示表元数据
\pset pager 0 查询结果显示不要进入交互模式
~~~
