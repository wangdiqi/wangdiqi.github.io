---
layout: post
title: "多线程服务器的常用实现"
subtitle: "多线程"
date: 2018-05-28 17:00:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - server
---

#### 高级的并发编程构件
TaskQueue
Producer-Consumer Queue
CountDownLatch等


#### 建议
1. RAII手法封装mutex的创建、销毁、加锁、解锁这四个操作；  
2. 只用非递归的mutex；  
3. 不手动调用lock()和unlock()函数，交给栈上的Guard对象。始终在同一个函数同一个scope里对某个mutexs加锁和解。避免在foo()里加锁，然后跑到bar()里解锁；也避免在不同的语句分支中分别加锁、解锁。
4. 每次构造Guard对象的时候，思考一路上(调用栈上)已经持有的锁，防止因加锁顺序不同而导致死锁。由于Guard对象是
栈上对象，看函数调用栈就能分析用锁的情况，非常便利。

5. 不使用跨进程的mutex，进程间通信只用TCP sockets
6. 加锁、解锁在同一线程，线程a不能去unlock线程b已经锁住的mutex(RAII自动保证)；
7. 别忘了解锁(RAII自动保证)
8. 不重复解锁(RAII自动保证)
9. 必要的时候可以考虑用PTHREAD_MUTEX_ERRORCHECK来排错。



#### 只使用非递归的mutex
错误情况：你以为拿到一个锁就能修改对象了，没想到外层代码已经拿到了锁，正在修改(或读取)同一个对象
~~~
MutexLock mutex;
std::vector<Foo> foos;

void post(const Foo &f)
{
    MutexLockGuard lock(mutex);
    foos.push_back(f);
}

void traverse()
{
    MutexLockGuard lock(mutex);
    for (std::vector<Foo>::const_iterator it = foos.begin(); it != foos.end(); ++it)
    {
        it->doit();
    }
}
~~~

将来有一天，Foo::doit() 间接调用了post()，那么会很有戏剧性的结果：
1. mutex是非递归的，于是死锁了；
2. mutex是递归的，又有push_back()可能(但不总是)导致vector迭代器失效，程序偶尔会有crash。

如果确实需要在遍历的时候修改vector，有两种做法：一是把修改推后，记住循环中试图添加或删除哪些元素，
等循环结束了再依记录修改foos；二是用copy-on-write;

如果一个函数既可能在已加锁的情况下调用，又可能在未加锁的情况下调用，那么就拆成两个函数
1. 跟原来的函数同名，函数加锁，转而调用第二个函数；
2. 给函数名加上后缀WithLockHold，不加锁，把原来的函数体搬过来；


~~~
void post(const Foo &f)
{
    MutexLockGuard lock(mutex);
    postWithLockHold(f);
}

void postWithLockHold(const Foo &f)
{
    foos.push_back(f);
}
~~~


## 参考
1. 潘爱民 <<Lock Convoys Explained>>
2. false sharing和CPU cache效应
3. https://nwcpp.org/talks/2007/Machine_Architecture_-_NWCPP.pdf
4. https://www.aristeia.com/TalkNotes/ACCU2011_CPUCaches.pdf
5. http://igoro.com/archive/gallery-of-processor-cache-effects/
6. http://simplygenius.net/Article/FalseSharing

