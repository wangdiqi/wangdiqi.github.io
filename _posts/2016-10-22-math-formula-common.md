---
layout: post
title: "数学公式"
subtitle: "基本算法使用到的数学公式"
date: 2016-10-22 12:00:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 数学
---

> "记载一些学习算法时使用到的一些数学公式，持续更新..."

### 模运算

* (a + b) % p = (a % p + b % p) % p

* (a - b) % p = (a % p - b % p) % p

* (a * b) % p = (a % p * b % p) % p

* (a^b) % p = ((a % p)^b) % p

* ((a+b) % p + c) % p = (a + (b+c) % p) % p

* ((a*b) % p * c)% p = (a * b*c) % p

* (a%p*b) % p= (a*b) % p

* (a + b) % p = (b+a) % p

* (a * b) % p = (b * a) % p

* ((a +b)% p * c) % p = ((a * c) % p + (b * c) % p) % p

* 若a≡b (% p)，则对于任意的c，都有(a + c) ≡ (b + c) (%p)

* 若a≡b (% p)，则对于任意的c，都有(a * c) ≡ (b * c) (%p)

* 若a≡b (% p)，c≡d (% p)，则 (a + c) ≡ (b + d) (%p)，(a - c) ≡ (b - d) (%p)，(a * c) ≡ (b * d) (%p)，(a / c) ≡ (b / d) (%p)
