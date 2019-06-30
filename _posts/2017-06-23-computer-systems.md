---
layout: post
title: "深入理解计算机系统"
subtitle: "深入理解计算机系统"
date: 2017-06-23 10:50:00
author: "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 深入理解计算机系统
---



## gcc
~~~
C版本                            GCC命令行选项

GNU 89                          无,-std=gnu89
ANSI, ISO C90                   -ansi, -std=c89
ISO C99                         -std=c99
ISO C11                         -std=c11
~~~

## PRI
printf("x= %" PRId32 ", y= %" PRIu64 "\n", x, y)
|
|
printf("x= %d, y=%lu\n", x, y)


## 补码

32位程序上C语言整型数据类型的典型取值范围
~~~
C数据类型                    最小值                    最大值

char                        -128                      127
unsigned char               0                         255

short                       -32768                    32767
unsigned short              0                         65535

int
unsigned

long
unsigned long

int32_t
uint32_t

int64_t
uint64_t
~~~


%d  以符号十进制
%u  无符号十进制
%x  十六进制

整数的数据与算术操作术语。下标w表示数据表示中的位数
~~~
B2Tw               二进制补码                    -Xw-1 * 2^w-1 + （(Xi*2i)  0<= i <=w-2）求和
B2Uw               二进制转无符号数               （Xi * 2^i）求和   0<= i <=w-1
U2Bw               无符号数转二进制
U2Tw               无符号转补码                   1. u <= TMaxw; u 2. u > TMaxw; u - 2^w
T2Bw               补码转二进制
T2Uw               补码转无符号数                  1. x < 0; x+2^w  2. x>=0; x
TMinw              最小补码值                    -2^w-1
TMaxw              最大补码值                    2^w-1 - 1
UMaxw              最大无符号数                   2^w - 1
~~~



** 截断无符号数 **
x' = x mod 2^k

** 截断补码数值 **
x' = U2Tk(x mod 2^k)

** 无符号数加法 **

0 <= x, y < 2^w的x和y有：
1.x+y < 2^w;x + y
2. 2^w <= x+y < 2^(w+1); x+y-2w

** 无符号数加法溢出检查 **
x > sum 时发生溢出

** 无符号数求反 **
1. x = 0; x
2.x > 0; 2^w - x

** 补码加法 **
1. 2^(w-1) <= x + y 正溢出, x + y - 2^w

2. 正常

3. x + y < -2^(w-1), 负溢出， x + y + 2^w

** 检测补码加法中的溢出 **
当且仅当 x < 0, y < 0，但 s >= 0时，s发生了负溢出。当且仅当x > 0, y > 0，但s <= 0时，s发生了正溢出


** 补码的非 **
1. x = TMinw; TMinw
2. -x, x > TMinw

** 无符号乘法 **
x * y = (x * y) mod 2^w

** 补码乘法 **
x * y = U2Tw((x * y) mod 2^w)

** 无符号和补码乘法的位级等价性 **

** IEEE浮点数 **
IEEE符点标准用 V = (-1)^s * M * 2^E

符号 s决定符号

尾数 M是一个二进制小数，它的范围是1-2-u 或者0-1-u

阶码 E的作用是对浮点数加权，这个权重是2的E次幂(可能是负数)

单精度
~~~
31 | 30 23 |  22    0
s  |  exp  |   frac
~~~


1.规格化的， exp的位模式既不全0，也不全1

2.exp 全0

3.exp 全1
