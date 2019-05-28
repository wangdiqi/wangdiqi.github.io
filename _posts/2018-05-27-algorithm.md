---
layout: post
title: "算法"
subtitle: "常用算法"
date: 2018-05-27 17:50:00
author: "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 算法
---

## 经典问题
O(n) 在无序数组中找到第K大的数

## 堆排序

#### 定义
(二叉)堆是一个数组，它可以被看成一个近似的完全二叉树，除了最底层外，该树是完全充满的，而且是从左向右填充。  

表示堆的数组A包括两个属性：A.length(通常)给出数组元素的个数，A.heap_size表示有多少个堆元素存储在该数组中。  

对于结点i：  
PARENT(i):return i/2

LEFT(i):return 2*i

RIGHT(i):return 2*i + 1

#### 定理
1. 子数组A(n/2 + 1 .. n)中的元素都是树的叶结点  

2. 含n个元素的堆的高度为lgn(向下取整)

3. 在最大堆的任一子树中，该子树所包含的最大的最大元素在该子树的根结点上  


#### 应用
作为高效的优先队列

INSERT, MINIMUM, EXTRACT-MAX, INCREASE-KEY




## 二叉树

#### 定理
1.如果一颗二叉搜索树中的一个结点有两个孩子，那么它的后继没有左孩子，它的前驱没有右孩子  
2.若关键字互不相同，如果T中一个结点x的右子树为空，且x有一个后继y，那么y一定是x的最底层祖先，并且其左孩子也是x的祖先  

#### 遍历
1.中序遍历(inorder tree walk)：输出的子树根的关键字位于其左子树的关键字值和右子树的关键字值之间  

~~~
INORDER—TREE-WALK(x)
if x != NIL
    INORDER-TREE-WALK(x.left)
    print x.key
    INORDER-TREE-WALK(x.right)
~~~

2.先序遍历(preorder tree walk)：输出的子树根的关键字在其左右子树的关键字值之前


3.后序遍历(postorder tree walk)：输出的子树根的关键字在其左右子树的关键字值之后

#### 搜索
~~~
TREE-SEARCH(x, k)
if x == NIL or k == x.key
  return x

if k < x.key
  return TREE-SEARCH(x.left, k)
else return TREE-SEARCH(x.right, k)
~~~

~~~
TREE-MINIMUM(x)
while x.left != NIL
 x = x.left
return x
~~~

~~~
TREE-MINIMUM(x)
while x.right != NIL
 x = x.right
return x
~~~

#### 后继和前驱

~~~
TREE-SUCCESSOR(x)
if x.right != NIL
  return TREE-MINIMUM(x, right)

y = x.p
while y != NIL and x == y.right
  x = y
  y = y.p

return y
~~~

#### 插入

~~~
TREE-INSERT(T, z)
y = NIL
x = T.root
while x != NIL
  y = x
  if z.key < x.key
    x = x.left
  else
    x = x.right

z.p = x
if y == NIL
  T.root = z
elseif z.key < y.key
  y.left = z
else
  y.right = z
~~~


#### 删除
1.如果z没有孩子结点，那么简单将z删除，修改父结点
2.如果z只有一个孩子，将孩子提升到树中z的位置上，并修改z的父结点
3.如果z有两个孩子，那么找z的后继y(一定在z的右子树中)，并让y占据树中z的位置。z原来的右子树部分成为y的新的右子树，并且z的左子树成为y的新的左子树
  这种情况稍显麻烦，与y是否为z的右孩子有关  
  
  a.如果z没有左孩子，那么用其右孩子来替代z，这个右孩子可以是NIL，也可以不是。当z的右孩子是NIL时，这种情况归为z没有孩子结点。当z的右孩子不是nil时，
    这种情况就是z仅有一个孩子结点的情形，该孩子是其右孩子。
  b.如果z仅有一个孩子且为其左孩子，那么用左孩子来替换z
  c.否则，z既有一个左孩子又有一个右孩子，我们要查z的后继y，这个后继位于z的右子树中并且没有i左孩子。将y移出原来的位置进行拼接，并替换树中的z
  d.如果y是z的右孩子，那么用y替换z，仅留下y的右孩子
  e.否则，y位于z的右子树中但并不是z的右孩子。这种情况下，先用y的右孩子替换y，然后再用y替换z


移动子树的操作(用v为根的子树来替换一颗以u为根的子树，结点u的双亲就变为结点v的双亲，并且最后v成为u的双亲相应的孩子)
~~~
TRANSPLANT(T, u, v)
if u.p == NIL
  T.root = v
elseif u == u.p.left
  u.p.left = v
else u.p.right = v
if v != nil
  v.p = u.p
~~~

~~~
TREE-DELETE(T, z)
if z.left == NIL
  TRANSPLANT(T, z, z.right)
elseif z.right == NIL
  TRANSPLANT(T, z, z.left)
else y = TREE-MINIMUM(z,right)
  if y.p != z
    TRANSPLANT(T, y, y.right)
    y.right = z.right
    y.right.p = y
  TRANSPLANT(T, z, y)
  y.left = z.left
  y.left.p = y
~~~


## 红黑树

红黑树是一颗二叉搜索树，
