---
layout: post
title: C++ - shared_from_this가 가리키는 곳을 변경하지 못하도록 규정
published: true
categories: [C++]
tags: c++ c++17 shared_from_this
---
[출처](https://faithandbrave.hateblo.jp/entry/2016/05/27/133110 )  
  
this 포인터를 std::shared_ptr로 얻을 수 있는 기능으로 std::enable_shared_from_this 기본 클래스와 멤버 함수 shared_from_this()가 있다.  
 
std::enable_shared_from_this에서 파생된 클래스의 개체를 new 하고 shared_ptr의 생성자에 전달하면 이 개체의 this를 shared_ptr로 취득 할 수 있다. 
  
그러나 아래와 같은 포인터에서 복수의 shared_ptr 개체를 만든 경우 shared_from_this()로  반환된 shared_ptr가 어떤 객체를 가리키는 것인가 규정되어 있지 않았다.  
  
```
#include <memory>

using namespace std;

int main()
{
  struct X : public enable_shared_from_this<X> { };
  auto xraw = new X;
  shared_ptr<X> xp1(xraw);  // #1
  {
    shared_ptr<X> xp2(xraw, [](void*) { });  // #2
  }
  xraw->shared_from_this();  // #3
}
```
  
  
구현에 따라서는 #3가 xp1를 가리키는 경우도 있고, xp2를 가리키는 경우도 있었다. xp2를 가리키는 경우 shared_from_this()가 다시 속박이 행해지는 것을 의미한다.  
  
C++ 17에서는 shared_from_this()가 다시 속박 되지 않는 것이 기본이다(이 경우 xp1를 계속 가리킨다). 또한 this를 속박하는 동작이 스레드로부터 안전하지 않은 것도 명시 되어 있다.  
  
원래 미 규정이었던 동작이지만 xp2를 가르키는 것을 기대하고 있던 프로그램은 작동하지 않을 가능성이 있으므로 주의해야한다.  
  
또 다시 속박 되지 않는 스펙을 명확하게 하는 것을 주된 목적으로 std::enable_shared_from_this 클래스에 weak_ptr의 this를 반환하는 weak_from_this() 멤버 함수가 추가되었다.  
  