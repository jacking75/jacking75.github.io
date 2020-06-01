---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) Coercion by Member Template
published: true
categories: [C++]
tags: c++ idiom Coercion
---
클래스 B가 A에서 파생되어 B 타입 객체에 대한 포인터는 A 타입의 포인터에 대입할 수 있지만 이들 타입의 복합 타입은 바탕이 된 타입의 관계를 공유하지 않는다.  
  
예를 들면 template_class<B>는 template_class<A>에 대입할 수는 없다.  
  
```
class A{};
struct B:A{};
template<class T>
class template_class{};

A* aptr;
B* bptr;
aptr=bptr;

template_class<A> tca;
template_class<B> tcb;
tca=tcb; // ill-formed
```  
  
그러나 이를 허용함으로써 스마트 포인터와 같은 래퍼 클래스에서의 취급에서 유용하게 될 때가 있다. 이것을 실현하는 것이 Coercion by Member Template idiom 이다.  
  
```
class A{};
struct B:A{};

template<class T>
struct ptr_wrapper{
    ptr_wrapper():ptr(nullptr){}
    // ...
    template<class U>
    ptr_wrapper(const ptr_wrapper<U>& p):ptr(p.ptr){}
    template<class U>
    ptr_wrapper& operator=(const ptr_wrapper<U>& p)
    {
        ptr=p.ptr;
        return *this;
    }   
    T* ptr;
};
```  
  