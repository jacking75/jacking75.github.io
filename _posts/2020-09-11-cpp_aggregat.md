---
layout: post
title: C++ - Aggregates(집성체)
published: true
categories: [C++]
tags: c++ aggregat
---
[출처](http://nekko1119.hatenablog.com/entry/2015/04/21/021133 )  
  
## Aggregates(이하, 집성체)는 배열과 아래의 조건을 충족한 클래스이다.  
- 사용자 정의 생성자(복사, 무브 포함)이 없음.
    - 생성자는 전혀 기술되지 않거나 default 지정된 상태다.
- 모든 데이터 멤버가 public 이다. static 데이터 멤버의 가시성은 영향을 주지 않는다.
- 기본(부모) 클래스가 없음
- 가상 함수가 없음  
  
  
```
struct Base {};

// 집성체가 아니다
class B : Base { // 기본 클래스가 있다
    // public 이 아닌 데이터 멤버
    int a;

public:
    // 가상 함수
    virtual void f() {}
    // 유저 정의 생성자
    B(B const&) {}
};

// 집성체
struct A {
    double d;
    B b;
    void g() {}
};

// 집성체
B bs[3];
```  
  
  
## 주의점
집성체 데이터 멤버가 비 집성체라도 괜찮다. 가시성 이외에 데이터 멤버 자체에 요구되는 컨셉은 아니다   
데이터 멤버가 인라인 초기화를 가지고 있어도 괜찮다.   
이상으로 어느 클래스가 집성체를 만족할 때 TriviallyCopyable 나 TriviallyDefaultConstructible 가 아니고, StandardLayout 라는 법도 없다.    
  
```
#include <type_traits>

struct X {
    virtual void f() { }
};

struct aggregate {
    X x;
    int i;
};

int main()
{
    aggregate agg = { { }, 42 };
    
    static_assert(!std::is_trivially_copy_constructible<aggregate>{ }, "");
    static_assert(!std::is_trivially_default_constructible<aggregate>{ }, "");
    static_assert(!std::is_standard_layout<aggregate>{ }, "");
}
```  
   
 
  
 