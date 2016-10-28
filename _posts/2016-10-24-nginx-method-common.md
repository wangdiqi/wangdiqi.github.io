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

## ngx_queue_t 双向链表
~~~
/*将链表容器h初始化，这时会自动置为空链表
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_init(h)

/*检测链表容器中是否为空，即是否没有一个元素存在。若返回非0，表示链表h是空的
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_empty(h)

/*将元素x插入到链表容器h的头部
INPUT：
h -- 链表容器结构体ngx_queue_t的指针
x -- 插入元素结构体中ngx_queue_t成员的指针*/
ngx_queue_insert_head(h, x)

/*将元素x插入到链表容器h的末尾
INPUT：
h -- 链表容器结构体ngx_queue_t的指针
x -- 插入元素结构体中ngx_queue_t成员的指针*/
ngx_queue_insert_tail(h, x)

/*返回链表容器h中的第一个元素的ngx_queue_t结构体指针
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_head(h)

/*返回链表容器h中的最后一个元素的ngx_queue_t结构体指针
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_last(h)

/*返回链表容器结构体的指针
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_sentinel(h)

/*由容器中移除x元素
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_remove(x)

/*用于拆分链表，h是链表容器，q是链表h中的一个元素。这个方法将链表h以元素q为界拆分成两个链表h和n，
其中h由原链表的前半部分构成(不包括q)，而n由原链表的后半部分构成，q是它的首元素
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_split(h, q, n)

/*合并链表，将n链表添加到h链表的末尾
INPUT：
h -- 链表容器结构体ngx_queue_t的指针
n -- 另一个链表容器结构体ngx_queue_t的指针*/
ngx_queue_add(h, n)

/*返回链表中心元素，如，链表共有N个元素，ngx_queue_middle方法将返回第N/2+1个元素
INPUT：
h -- 链表容器结构体ngx_queue_t的指针*/
ngx_queue_middle(h)

/*使用插入排序法对链表进行排序，cmpfunc需要使用者自己实现，原型是：
ngx_int_t (*cmpfunc)(const ngx_queue_t *, const ngx_queue_t *)
INPUT：
h -- 链表容器结构体ngx_queue_t的指针
cmpfunc -- 两个链表元素的比较方法，如果返回正数，则表示以升序排列*/
ngx_queue_sort(h, cmpfunc)

对于链表中的每一个元素，其类型可以是任意的struct结构体，但结构体必须要有一个ngx_queue_t类型的成员。
当ngx_queue_t作为链表的元素成员使用时，有以下四个方法。

/*返回q元素的下一个元素
INPUT：
q -- 链表中某一个元素结构体的ngx_queue_t成员的指针*/
ngx_queue_next(q)

/*返回q元素的上一个元素
q -- 链表中某一个元素结构体的ngx_queue_t成员的指针*/
ngx_queue_prev(q)

/*返回q元素(ngx_queue_t类型)所属结构体(任何struct类型，其中可在任意位置包含ngx_queue_t类型的成员)的地址
q -- 链表中某一个元素结构体的ngx_queue_t成员的指针
type -- 链表元素的结构体类型名称(该结构体中必须包含ngx_queue_t类型的成员)
link -- 是上面这个结构体中ngx_queue_t类型的成员名字*/
ngx_queue_data(q, type, link)

/*将元素x插入到元素q之后
q -- 链表中某一个元素结构体的ngx_queue_t成员的指针
x -- 插入元素结构体中ngx_queue_t成员的指针*/
ngx_queue_insert_after(q, x)

~~~

## ngx_array_t 动态数组
~~~
~~~

## ngx_list_t 单向链表
~~~
~~~

## ngx_rbtree_t 红黑树
~~~
~~~

## ngx_radix_tree_t 基数树
~~~
~~~

## hash
~~~
/*
INPUT:
hash -- 散列表结构体指针
key -- 根据散列函数算出来的散列关键字
name -- 实际关键字的地址
len -- 实际关键字的长度
OUTPUT:
返回散列表中关键字与name、len指定关键字完全想同的槽中，ngx_hash_elt_t结构体中value成员所指向的用户数据，若没有
返回NULL*/
void *ngx_hash_find(ngx_hash_t *hash, ngx_uint_t key, u_char *name, size_t len)

/*散列方法指针，通过这个可以自定义散列方法
INPUT：
data -- 元素关键字首地址
len -- 元素关键字的长度
OUTPUT:
返回散列值*/
typedef ngx_uint_t (*ngx_hash_key_pt)(u_char *data, size_t len);

nginx提供了两种基本的散列方法，它会假定关键字是字符串,如下:
/*使用BKDR算法将任意长度的字符串映射为整型*/
ngx_uint_t ngx_hash_key(u_char *data, size_t len)
/*将字符串全小写后，再使用BKDR算法将任意长度的字符串映射为整型*/
ngx_uint_t ngx_hash_key_lc(u_char *data, size_t len)

nginx提供了两种查询前置、后置通配符散列表的方法：
/*将待查询关键字name转换为前置散列表规则下的字符串再递归查询，成功时返回找到元素指向的用户数据，否则NULL*/
void *ngx_hash_find_wc_head(ngx_hash_wildcard_t *hwc, u_char *name, size_t len)
/*将待查询关键字name转换为后置散列表规则下的字符串再递归查询，成功时返回找到元素指向的用户数据，否则NULL*/
void *ngx_hash_find_wc_tail(ngx_hash_wildcard_t *hwc, u_char *name, size_t len)
/*
INPUT:
key -- 根据name算出的hash值*/
void *ngx_hash_find_combined(ngx_hash_combined_t *hash, ngx_uint_t key, u_char *name, size_t len);

ngx_hash_init_t提供的两个初始化方法：
/*初始化基本的散列表。返回NGX_OK，表示初始化成功，这时names数组已经添加到hinit->hash散列表中了；NGX_ERROR表示初始化失败。
INPUT:
hinit -- 散列表初始化结构体的指针
names -- 数组的首地址,这个数组中每个元素以ngx_hash_key_t作为结构体，它存储着预添加到散列表中的元素
nelts -- names数组的的元素数目
OUTPUT:*/
ngx_int_t ngx_hash_init(ngx_hash_init_t *hinit, ngx_hash_key_t *names, ngx_uinit_t nelts)

/*初始化通配符散列表(前置或者后置)。返回NGX_OK，表示初始化成功，这时names数组已经添加到hinit->hash散列表中了；NGX_ERROR表示初始化失败
hinit -- 散列表初始化结构体的指针
names -- 数组的首地址，这个数组中每个元素以ngx_hash_key_t作为结构体，它存储着预添加到散列表中的元素(这些元素的关键字要么含有前置通配符，要么含有后置通配符)；
nelts -- nelts是names数组的元素数目*/
ngx_int_t ngx_hash_wildcard_init(ngx_hash_init_t *hinit, ngx_hash_key_t *names, ngx_uint_t nelts)

ngx_hash_keys_arrays_t提供的两个方法：
/*初始化ngx_hash_keys_arrays_t结构体，在向ha加入成员前必须先调用该方法。NGX_OK,NGX_ERROR
ha -- 要初始化的ngx_hash_keys_arrays_t结构体指针
type -- 取值范围有两个，其中NGX_HASH_SMALL表示待初始化的元素较少，而NGX_HASH_LARGE表示待初始化的元素较多*/
ngx_ini_t ngx_hash_keys_array_init(ngx_hash_keys_arrays_t *ha, ngx_uint_t type)

/*向ha中添加1个元素。NGX_OK,NGX_ERROR
ha -- 要初始化的ngx_hash_keys_arrays_t结构体指针
key -- 添加元素的关键字
value -- key关键字对应的用户数据的指针
flags -- 取值有3种：NGX_HASH_WILDCARD_KEY表示需要处理通配符；NGX_HASH_READONLY_KEY表示关键字不可以做
         更改(也就是不可以通过全小写关键字来获取散列码)；其他值表示既不处理通配符，又允许通过把关键字全小写
         来获取散列码*/
ngx_int_t ngx_hash_add_key(ngx_hash_keys_arrays_t *ha, ngx_str_t *key, void *value, ngx_uint_t flags)

~~~

## 宏
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
