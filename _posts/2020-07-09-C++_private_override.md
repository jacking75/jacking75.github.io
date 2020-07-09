---
layout: post
title: C++ - 자식 클래스에서 오버라이드한 멤버함수의 접근 지정이 부모 클래스의 멤버함수와 다른 경우
published: true
categories: [C++]
tags: c++ virtual
---
  
```
class Base
{
public :
    virtual void f() { } 
} ;

class Derived : public Base
{
private :
    void f() { } 
} ;

int main()
{
    Derived d ;
    d.f();   // 1. ?

    Base & ref = d ;
    ref.f();  // 2. ? 

    return 0;
}
```  
  
1번은 에러.  
  
2번 virtual 함수의 접근 조사는 호출 시점에서의 선언에 의해 정적으로 결정된다. 그래서 main()에서 Base::f로는 접근할 수 있으므로  ref.f()는 에러가 되지 않으며 virtual 함수 호출에 의해 Derived의 f를 호출한다.  
  