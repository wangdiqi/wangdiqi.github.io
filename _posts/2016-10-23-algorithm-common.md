---
layout:     post
title:      "常用算法"
subtitle:   "简短的常用算法"
date:       2016-10-23 21:49:00
author:     "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 算法
---

### 模幂运算

~~~
inline unsigned __int64 MulMod(unsigned __int64 a, unsigned __int64 b, unsigned __int64 n)
{
  return a * b % n;
}

/*
模幂运算，返回值 x=base^pow mod n
*/
unsigned __int64 PowMod(unsigned __int64 base, unsigned __int64 pow, unsigned __int64 &n)
{
  unsigned __int64 a=base, b=pow, c=1;
  while(b)
  {
    while(!(b & 1))
    {
      b>>=1; //a=a * a % n; //
      a=MulMod(a, a, n);
    } b--; //c=a * c % n; //
      c=MulMod(a, c, n);
  }
  
  return c;
}
~~~

### 判断素数

~~~
#include<math.h>
bool IsPrime(unsigned n)
{
  unsigned maxFactor = sqrt(n); //n的最大因子
  for (unsigned i = 2 ; i <= maxFactor ; i++)
  {
    if (!(n % i)) //n能被i整除，则说明n非素数
    return false;
  }

  return true;
}
~~~

### 最大公约数

~~~

//递归
unsigned Gcd(unsigned a , unsigned b)
{
  if (b)
  return Gcd(b , a % b);
  return a;
}


//迭代
unsigned Gcd(unsigned a , unsigned b)
{
  unsigned temp;
  while (b)
  {
    temp = a % b;
    a = b;
    b = temp;
  }

  return a;
}
~~~
