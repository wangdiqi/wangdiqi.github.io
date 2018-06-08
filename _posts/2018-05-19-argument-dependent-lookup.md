---
layout: post
title: "Argument-dependent lookup(ADL, or Koenig lookup)"
subtitle: "C++"
date: 2018-05-19 23:05:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - C++
---

## 定义

[Argument-dependent lookup(ADL, or Koenig lookup)](http://en.cppreference.com/w/cpp/language/adl)

## 说明
Koenig lookup 主要是编译器帮助我们在相关的命名空间内寻找依赖并且符合的API

## 例子

#### 最简单的例子

~~~
namespace A
{
  void f(int){};
}

namespace B
{
  void f(int){};
  void test()
  {
    f(1);                //这会调用namespace B下的f(int)
  }
}
~~~

现在我们在namespace A下新增一个class类型Person，并将f(int) 修改为 f(Person &)

~~~
namespace A
{
  class Person{};
  void f(Person &){};
}

namespace B
{
  void f(A::Person &){};
  void test()
  {
    A::Person p;
    f(p);                //ambiguous
  }
}
~~~

为什么会ambiguous？首先，在test中，B::f 一定是可见的，如果冲突，一定是A::f也被加入了候选项目。这就是ADL自动帮我们做的事：
因为f(p)调用时，因为参数p是namespace A下的类型，ADL会自动搜索参数所在的namespace。

现在，我们再换个例子，将B下的nonmember function 换成 member function

~~~
namespace A
{
  class Person{};
  void f(Person &){};
}

namespace B
{
  class Obj
  {
    void f(A::Person &){};
    void test()
    {
      A::Person p;
      f(p);              //ok
    }
  };
}
~~~

现在，可编译通过。为什么？因为member function 比 nonmember function 拥有更高的优先级。


## 实际分析一个问题
问题来源于 C++标准库(第二版)812页：针对“命名空间内的类型”而写出的用户自定义 operator << 会有些局限。原因是它将无法在ADL情境下被找到。例如

~~~
template <typename T1, typename T2>
std::ostream & operator << (std::ostream &os, const std::pair<T1, T2> &p)
{
  return os << p.first << p.second;
}

int main(int argc, char **argv)
{
  std::pair<int, long> p(1, 2);
  std::cout << p << std::endl; // (1)

  std::vector<std::pair<int, long>> v;
  std::copy(v.begin(), v.end(), std::ostream_iterator<std::pair<int, long>>(std::cout, "\n"));          // (2)这里，编译会报错

  return 0;
}
~~~

现在，我们来分析一下这个问题：  
(1)处，调用operation <<, 会在namespace std下和当前的namespace下(global)查找该function，找到的是我们global 下的overload function

(2) 此处代码解释开应该是  

~~~
namespace std{
    maybeInOneFunction(...)
    {
      // ... do somethings

      std::pair<int, long> pairValue;
      std::out << pairValue;

      // ... do somethings
    }
}
~~~

   这时候，调用operator << 只会在namespace std下查找，因为当前的namespace和参数的namespace都是std  
   
应该怎么样修改？举个简单例子，不要直接使用std::pair作为function的参数，而是将其包裹一层作为参数(创建一个类)  

~~~
template<typename T1, typename T2>
class WrapperPair
{
 public:
  WrapperPair(std::pair<T1, T2> value) :value_(value){}
  std::pair<T1, T2> value_;
};

template <typename T1, typename T2>
std::ostream & operator << (std::ostream &os, const WrapperPair<T1, T2> &p)
{
  return os << p.value_.first << p.value_.second;
}

int main(int argc, char **argv)
{
  // std::pair<int, long> p(1, 2);
  // std::cout << p << std::endl;

  std::vector<WrapperPair<int, long>> v;
  v.push_back(WrapperPair<int, long>(std::make_pair(1,2)));
  std::copy(v.begin(), v.end(), std::ostream_iterator<WrapperPair<int, long>>(std::cout, "\n"));

  return 0;
}
~~~

因为WrapperPair是在global下，所以我们overload的operator << function可以找到

上述修改例子有更优雅的实现，详见：[elegant implement](http://www.cplusplus.com/forum/general/224129/)

## 参考

[http://www.gotw.ca/publications/mill02.htm](http://www.gotw.ca/publications/mill02.htm)
