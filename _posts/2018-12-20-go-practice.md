---
layout: post
title: "go"
subtitle: "go"
date: 2018-12-20 09:50:00
author: "seventh"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - go
---

> "Let's go"

## 常用库
"io"
"ioutil"
"bufio"
"bytes"
"strings"
"context"


## 最佳实践

go编程中的准则：

第一条原则是，绝对不能由消费者关channel，因为向关闭的channel写数据会panic。正确的姿势是生产者写完所有数据后，关闭channel，消费者负责消费完channel里面的全部数据  

为什么consume要读完channel里面所有数据？因为go produce()可能有多个，这样写的代码，在读完ch可以确定所有produce的goroutine都退出了，不会泄漏。  
~~~
func produce(ch chan<- T) {
    defer close(ch) // 生产者写完数据关闭channel
    ch <- T{}
}
func consume(ch <-chan T) {
    for _ = range ch { // 消费者用for-range读完里面所有数据
    }
}
ch := make(chan T)
go produce(ch)
consume(ch)
~~~


第二条原则是，利用关闭channel来广播取消动作。向关闭的channel读数据永远不会阻塞，这是进阶的技巧。假设消费者拿到数据处理后有error发生，整个动作失败，那么需要有某种机制通知生产者停止并退出。

~~~
func produce(ch chan<- T, cancel chan struct{}) {
    select {
      case ch <- T{}:
      case <- cancel: // 用select同时监听cancel动作
    }
}
func consume(ch <-chan T, cancel chan struct{}) {
    v := <-ch
    err := doSomeThing(v)
    if err != nil {
        close(cancel) // 能够通知所有produce退出
        return
    }
}
for i:=0; i<10; i++ {
    go produce()
}
consume()
~~~

WaitGroup之类的可以配合着用
context里面的cancel比较值得参考和学习 
