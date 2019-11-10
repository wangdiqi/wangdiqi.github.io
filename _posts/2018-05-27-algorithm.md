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
## 导论
10个数据结构：数组、链表、栈、队列、散列表、二叉树、堆、跳表、图、Trie树  
10个算法：递归、排序、二分查找、搜索、哈希算法、贪心算法、分治算法、回朔算法、动态规划、字符串匹配算法


## 复杂度分析

大 O 复杂度表示法

所有代码的执行时间T(n)与每行代码的执行次数n成正比

~~~
T(n) = O(f(n))

n表示数据规模的大小
f(n)表示每行代码执行的次数总和
O表示代码的执行时间T(n)与f(n)表达式成正比
~~~

1. 只关注循环执行次数最多的一段代码
2. 加法法则：总复杂度等于量级最大的那段代码的复杂度
~~~
T1(n) = O(f(n)), T2(n) = O(g(n));　
那么T(n) = T1(n) + T2(n) = max(O(f(n)), O(g(n))) = O(max(f(n), g(n)))
~~~
3. 乘法法则：嵌套代码的复杂度等于嵌套内外代码复杂度的h乘积
~~~
如果T1(n) = O(f(n)), T2(n) = O(g(n));
那么T(n) = T1(n) * T2(n) = O(f(n)) * O(g(n)) = O(f(n) * g(n))
~~~

~~~
常量阶 O(1)
对数阶 O(logn)
线性阶 O(n)
线性对数阶 O(nlogn)
平方阶 O(n^2)
立方阶 O(n^3)
K次方阶 O(n^k)
指数阶 O(2^n)
阶乘阶 O(n!)
~~~

### 多项式量级

### 非多项式量级
O(2^n)
O(n!)
当数据规模n越来越大时，非多项式量级算法的执行时间会急剧增加

### NP问题

### 复杂度分析例子

~~~
int i = 8;
int j = 6;
int sum = i + j

O(1)
~~~

~~~
i = 1;
while (i <= n) {
    i = i * 2;
}

i * 2^k = n
k = log(n/i)
O(logn)

实际上，不管是以2为底，以3为底，还是以10为底，我们可以把所有对数阶的时间复杂度都记为O(logn),因为：
对数之间是可以互相转换的， log3底n就等于log3底2 * log2底n

~~~

~~~
int cal(int m, int n) {
  int sum_1 = 0;
  int i = 1;
  for (; i < m; ++i) {
    sum_1 = sum_1 + i;
  }

  int sum_2 = 0;
  int j = 1;
  for (; j < n; ++j) {
    sum_2 = sum_2 + j;
  }

  return sum_1 + sum_2;
}

m和n是表示两个数据规模。我们无法事先评估m和n谁的量级大，所以我们在表示复杂度的时候，就不能简单地利用加法法则，省略其中一个。
所以，上面代码的时间复杂度就是O(m+n)

针对这种情况，原来的加法法则就不正确了，我们需要将加法法则改为：
T1(m) + T2(n) = O(f(m) + g(n))。但是乘法法则继续有效：
T1(m) * T2(n) = O(f(m) * f(n))

~~~

### 最好、最坏情况时间复杂度
~~~
// n 表示数组 array 的长度
int find(int[] array, int n, int x) {
  int i = 0;
  int pos = -1;
  for (; i < n; ++i) {
    if (array[i] == x) {
       pos = i;
       break;
    }
  }
  return pos;
}

要查找的变量x在数组中的位置，有n+1种情况：在数组的0 ～ n-1位置中和不在数组中。
我们把每种情况下，查找需要便利的元素个数累加起来，然后除以n+1，就可以得到需要遍历元素个数的平均值：
(1+2+3+....+n+n) / (n + 1) = n(n+3) / 2(n+1)
但是，该推导方式有个最大的问题是，没有将各种情况发生的概率考虑进去。如果我们把每种情况发生的概率也考虑进去，那平均时间复杂度的计算过程如下：
假设在数组中与不在数组中的概率都为1/2.另外查找的数据出现在0 ~ n-1这n个位置的概率也是一样的，为1/2。
所以根据概率乘法法则，要查找的数据u出现在0 ~ n-1 中任意位置的概率就是1/(2n).
1*1/2n + 2*1/2n + 3*1/2n + ... + n*1/2n + n*/2 = (3n + 1) / 4
这个值就是概率论中的加权平均值，也叫作期望值，所以平均时间复杂度的全称应该叫加权平均时间复杂度或者期望时间复杂度
~~~

### 均摊时间复杂度
~~~
 // array 表示一个长度为 n 的数组
 // 代码中的 array.length 就等于 n
 int[] array = new int[n];
 int count = 0;
 
 void insert(int val) {
    if (count == array.length) {
       int sum = 0;
       for (int i = 0; i < array.length; ++i) {
          sum = sum + array[i];
       }
       array[0] = sum;
       count = 1;
    }

    array[count] = val;
    ++count;
 }
最好情况时间复杂度为O(1)
最坏情况O(n)
平均O(1):假设数组的长度是n，根据数据插入的位置的不同，我们可以分为n种情况，每种情况的时间复杂度是O(1)。
初次之外，还有一种“额外”的情况，就是数组没有空闲空间时插入一个数据，这个时候的时间复杂度是O(n)。而且，
这n+1中n情况发生的概率一样，都是1/(n+1)。所以，根据加权平均的计算方法，我们求得平均时间复杂度就是：
1*1/(n+1) + 1*1/(n+1) + ... + 1*1/(n+1) + n*1/(n+1) = O(1)

这个例子有个区别，可以进行摊还分析法：
1. 大部分情况下的复杂度都是O(1)，个别情况下为O(n)
2. O(1)和O(n)的插入，出现的频率非常有规律，而且有一定的前后时序关系，一般都是一个O(n)插入之后，紧跟着n-1个O(1)的插入操作，循环往复。
所以我们把每一次O(n)的插入操作，都会跟着n-1次O(1)的插入操作，所以把耗时多的那次操作均摊到接下来的n-1次耗时少的操作下来
~~~

### 例题
~~~
// 全局变量，大小为 10 的数组 array，长度 len，下标 i。
int array[] = new int[10]; 
int len = 10;
int i = 0;

// 往数组中添加一个元素
void add(int element) {
   if (i >= len) { // 数组空间不够了
     // 重新申请一个 2 倍大小的数组空间
     int new_array[] = new int[len*2];
     // 把原来 array 数组中的数据依次 copy 到 new_array
     for (int j = 0; j < len; ++j) {
       new_array[j] = array[j];
     }
     // new_array 复制给 array，array 现在大小就是 2 倍 len 了
     array = new_array;
     len = 2 * len;
   }
   // 将 element 放到下标为 i 的位置，下标 i 加一
   array[i] = element;
   ++i;
}

最好情况时间复杂度为O(1)
最坏情况时间复杂度为O(n)
平均情况复杂度为O(1)
~~~

### 空间复杂度

## 线性表
数组
队列
链表
栈

## 非线性表
树
图

### 链表常见问题
单链表反转
链表中环的检测
两个有序的链表合并
删除链表倒数第n个结点
求链表的中间结点

普通问题：
单链表
循环链表
双向链表
单链表回文
LRU

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


## hash

装载因子 = 已存个数 / 总个数


## 图

#### 深度优先搜索

1) For an unweighted graph, DFS traversal of the graph produces the minimum spanning tree and all pair shortest path tree.

2) Detecting cycle in a graph
A graph has cycle if and only if we see a back edge during DFS. So we can run DFS for the graph and check for back edges. (See this for details)

3) Path Finding
We can specialize the DFS algorithm to find a path between two given vertices u and z.
i) Call DFS(G, u) with u as the start vertex.
ii) Use a stack S to keep track of the path between the start vertex and the current vertex.
iii) As soon as destination vertex z is encountered, return the path as the
contents of the stack

See this for details.

4) Topological Sorting
Topological Sorting is mainly used for scheduling jobs from the given dependencies among jobs. In computer science, applications of this type arise in instruction scheduling, ordering of formula cell evaluation when recomputing formula values in spreadsheets, logic synthesis, determining the order of compilation tasks to perform in makefiles, data serialization, and resolving symbol dependencies in linkers [2].

5) To test if a graph is bipartite
We can augment either BFS or DFS when we first discover a new vertex, color it opposited its parents, and for each other edge, check it doesn’t link two vertices of the same color. The first vertex in any connected component can be red or black! See this for details.

6) Finding Strongly Connected Components of a graph A directed graph is called strongly connected if there is a path from each vertex in the graph to every other vertex. (See this for DFS based algo for finding Strongly Connected Components)

7) Solving puzzles with only one solution, such as mazes. (DFS can be adapted to find all solutions to a maze by only including nodes on the current path in the visited set.)


#### 广度优先搜索

1) Shortest Path and Minimum Spanning Tree for unweighted graph In an unweighted graph, the shortest path is the path with least number of edges. With Breadth First, we always reach a vertex from given source using the minimum number of edges. Also, in case of unweighted graphs, any spanning tree is Minimum Spanning Tree and we can use either Depth or Breadth first traversal for finding a spanning tree.

2) Peer to Peer Networks. In Peer to Peer Networks like BitTorrent, Breadth First Search is used to find all neighbor nodes.

3) Crawlers in Search Engines: Crawlers build index using Breadth First. The idea is to start from source page and follow all links from source and keep doing same. Depth First Traversal can also be used for crawlers, but the advantage with Breadth First Traversal is, depth or levels of the built tree can be limited.

4) Social Networking Websites: In social networks, we can find people within a given distance ‘k’ from a person using Breadth First Search till ‘k’ levels.

5) GPS Navigation systems: Breadth First Search is used to find all neighboring locations.

6) Broadcasting in Network: In networks, a broadcasted packet follows Breadth First Search to reach all nodes.

7) In Garbage Collection: Breadth First Search is used in copying garbage collection using Cheney’s algorithm. Refer this and for details. Breadth First Search is preferred over Depth First Search because of better locality of reference:

8) Cycle detection in undirected graph: In undirected graphs, either Breadth First Search or Depth First Search can be used to detect cycle. In directed graph, only depth first search can be used.

9) Ford–Fulkerson algorithm In Ford-Fulkerson algorithm, we can either use Breadth First or Depth First Traversal to find the maximum flow. Breadth First Traversal is preferred as it reduces worst case time complexity to O(VE2).

10) To test if a graph is Bipartite We can either use Breadth First or Depth First Traversal.

11) Path Finding We can either use Breadth First or Depth First Traversal to find if there is a path between two vertices.

12) Finding all nodes within one connected component: We can either use Breadth First or Depth First Traversal to find all nodes reachable from a given node.
