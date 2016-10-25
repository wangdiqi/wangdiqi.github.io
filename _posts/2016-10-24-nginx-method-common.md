---
layout: post
title: "Nginx 基本函数"
subtitle: ""
date: 2016-10-24 14:00:00
author: seventhking
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - nginx
---

> "内容来自书本[^1]和网络"

### hash
~~~
void *ngx_hash_find(ngx_hash_t *hash, ngx_uint_t key, u_char *name, size_t len)
~~~

### 宏
~~~
1. #define offsetof(type, member) (size_t)&(((type *)0)->member)
2. #define ngx_tolower(c)      (u_char) ((c >= 'A' && c <= 'Z') ? (c | 0x20) : c)
3. #define ngx_toupper(c)      (u_char) ((c >= 'a' && c <= 'z') ? (c & ~0x20) : c)
~~~

<br/>
<br/>
<br/>

### 附录

[^1]:深入理解Nginx模块开发与架构解析(第2版) 陶辉 著
