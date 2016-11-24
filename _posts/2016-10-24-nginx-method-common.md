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
/*创建1个动态数组，并预分配n个大小为size的内存空间
INPUT:
p -- 内存池对象
n -- 初始分配元素的最大个数
size -- 每一个元素所占用的内存大小
*/
ngx_array_create(ngx_pool_t *p, ngx_uint_t n, size_t size)

/*初始化1个已经存在的动态数组，并预分配n个大小为size的内存空间
INPUT:
a -- 一个动态数组结构体指针
p -- 内存池
n -- 初始分配元素的最大个数
size -- 每一个元素所占用的内存大小
*/
ngx_array_init(ngx_array_t *a, ngx_pool_t *p, ngx_uint_t n, size_t size)

/*销毁已经分配的数组元素空间和ngx_array_t动态数组对象。
注意：ngx_array_destroy最好与ngx_array_create配对使用，
因为ngx_array_destroy同时会回收ngx_array_t结构体自身占用的内存
INPUT:
a -- 一个动态数组结构体指针
*/
ngx_array_destroy(ngx_array_t *a)

/*向当前a动态数组中添加1个元素，返回的是这个新添加元素的地址。注意：如果
动态数组已经达到容量上限，这时会自动扩容，扩容机制有两种情形：
(1) 如果当前内存池中剩余的空间大于或者等于本次需要新增的空间，那么本次扩容
    将只扩充新增的空间。例如，对于ngx_array_push方法来说，就是扩充1个元素，
    而对于ngx_array_push_n方法来说，就是扩充n个元素。
(2) 如果当前内存池中剩余的空间小于本次需要新增的空间，那么对ngx_array_push方法
    来说，会将原先动态数组的容量扩容一倍，而对于ngx_array_push_n来说，情况更
    复杂些，如果参数n小于原先动态数组的容量，将会扩容一倍；如果参数n大于原先动态
    数组的容量，这时会分配2 x n大小的空间，扩容会超过一倍。（注意开辟全新内存时，
    存在拷贝数据到新内存的过程，可能会很耗时）
INPUT:
a -- 一个动态数组结构体指针
*/
ngx_array_push(ngx_array_t *a)

/*向当前a动态数组中添加n个元素，返回的是新添加这批元素中第一个元素的地址
INPUT:
a -- 一个动态数组结构体指针
n -- 需要添加元素的个数
*/
ngx_array_push_n(ngx_array_t *a, ngx_uint_t n)
~~~

## ngx_list_t 单向链表
~~~
ngx_list_t *ngx_list_create(ngx_pool_t *pool, ngx_uint_t n, size_t size);

void *ngx_list_push(ngx_list_t *list);
~~~

## ngx_rbtree_t 红黑树
~~~
#define ngx_rbt_black(node)             ((node)->color = 0)

#define ngx_rbtree_sentinel_init(node)  ngx_rbt_black(node)

#define ngx_rbtree_init(tree, s, i)                                           \
    ngx_rbtree_sentinel_init(s);                                              \
    (tree)->root = s;                                                         \
    (tree)->sentinel = s;                                                     \
    (tree)->insert = i

void ngx_rbtree_insert(ngx_rbtree_t *tree, ngx_rbtree_node_t *node);

void ngx_rbtree_delete(ngx_rbtree_t *tree, ngx_rbtree_node_t *node);

static ngx_inline ngx_rbtree_node_t *
ngx_rbtree_min(ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel)
~~~

## ngx_radix_tree_t 基数树
~~~
ngx_radix_tree_t *ngx_radix_tree_create(ngx_pool_t *pool,
    ngx_int_t preallocate);

ngx_int_t ngx_radix32tree_insert(ngx_radix_tree_t *tree,
    uint32_t key, uint32_t mask, uintptr_t value);
    
ngx_int_t ngx_radix32tree_delete(ngx_radix_tree_t *tree,
    uint32_t key, uint32_t mask);
uintptr_t ngx_radix32tree_find(ngx_radix_tree_t *tree, uint32_t key);

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

## ngx_pool_t 内存池

*1.内存池操作*  
  
~~~
/*创建内存池，注意它的size参数并不等同于可分配空间，它同时包含了管理结构的大小，这意味着：size绝不能小于sizeof(ngx_pool_t)，
否则就会有内存越界错误。通常，可以设size为NGX_DEFAULT_POOL_SIZE，该宏目前为16KB，不用担心16KB会不够用，当第一个16KB用完时，
会自动再分配16KB内存*/
ngx_create_pool()

/*销毁内存池，它同时会把通过该pool分配出的内存释放，而且，还会执行通过ngx_pool_cleanup_add方法添加的各类资源清理方法*/
ngx_destroy_pool()

/*重置内存池，即将内存池中的原有内存释放后继续使用。这个方法的实现是，会把大块内存释放给操作系统，而小块内存则在不释放的情况下复用*/
ngx_reset_pool()
~~~

*2.基于内存池的分配、释放内存操作*  
  
~~~
/*分配地址对齐的内存。按总线长度(例如sizeof(unsigned long))对齐地址后，可以减少CPU读取内存的次数，当然代价是有一些内存浪费*/
ngx_palloc()

/*分配内存时不进行地址对齐*/
ngx_pnalloc()

/*分配出地址对齐的内存后，再调用memset将这些内存全部清0*/
ngx_pcalloc()

/*按参数alignment进行地址对齐来分配内存。注意，这样分配出的内存不管申请的size有多小，都是不会使用小块内存池的，
它会从进程的堆中分配内存，并挂在大块内存组成的large单链表中*/
ngx_pmemalign()

/*提前释放大块内存。它的效率不高，其实现是遍历large链表，寻找ngx_pool_large_t的alloc成员等与待释放地址，找到后
释放内存给操作系统，将ngx_pool_large_t移出链表并删除*/
ngx_pfree()
~~~

*3.随着内存池释放同步释放资源的操作*  
  
~~~
/*添加一个需要在内存池释放时同步释放的资源。该方法会返回一个ngx_pool_cleanup_t结构体，而我们得到后需要设置
ngx_pool_cleanup_t的handler成员为释放资源时执行的方法。ngx_pool_cleanup_add有一个参数size，当它不为0
时，会分配size大小的内存，并将ngx_pool_cleanup_t的data成员指向该内存，这样可以利用这段内存传递参数，供
释放资源的方法使用。当size为0时，data将为NULL*/
ngx_pool_cleanup_add()

/*在内存池释放前，如果需要提前关闭文件(当然是调用过ngx_pool_cleanup_add添加的文件，同时ngx_pool_cleanup_t
的handler成员被设为ngx_pool_cleanup_file)，则调用该方法*/
ngx_pool_run_cleanup_file()

/*以关闭文件来释放资源的方法，可以设置到ngx_pool_cleanup_t的handler成员*/
ngx_pool_cleanup_file()

/*以删除文件来释放资源的方法，可以设置到ngx_pool_cleanup_t的handler成员*/
ngx_pool_delete_file()
~~~

*与内存池无关的分配、释放操作*  
  
~~~
/*从操作系统中分配内存*/
ngx_alloc()

/*从操作系统中分配出内存，再调用memset把内存清0*/
ngx_calloc()

/*释放内存到操作系统*/
ngx_free()
~~~

## 宏
~~~
1. #define offsetof(type, member)                 \
       (size_t)&(((type *)0)->member)

2. #define ngx_tolower(c)                         \
       (u_char) ((c >= 'A' && c <= 'Z') ? (c | 0x20) : c)

3. #define ngx_toupper(c)                         \
       (u_char) ((c >= 'a' && c <= 'z') ? (c & ~0x20) : c)

4. 寻找最近的对齐地址
   #define ngx_align_ptr(p, a)                    \
       (u_char *) (((uintptr_t)(p) + ((uintptr_t) a - 1)) & ~((uintptr_t) a - 1))

5. 查询双向链表数据地址
   #define ngx_queue_data(q, type, link) \
       (type *)((u_char *) q - offsetof(type, link))
~~~

<br/>
<br/>
<br/>

### 附录

[^1]:深入理解Nginx模块开发与架构解析(第2版) 陶辉 著
