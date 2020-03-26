---
layout: post
title: C++17 - atomic<T>::is_always_lock_free
published: true
categories: [C++]
tags: c++ c++17 atomic is_always_lock_free
---
C++ 17은 atomic 클래스 템플릿에 is_always_lock_free라는 정적 멤버 상수가 정의 되어 있는데 C++11의 `is_lock_free()`의 static 멤버 버전이라고 생각하면 좋다.   
   
`is_lock_free()`를 사용하기 위해서는 atomic 클래스 오브젝트를 만들어야 하지만 `is_always_lock_free`는 만들지 않는다.  
  
```
namespace std {
     template < class T>
     class atomic {
     public :
         static constexpr bool is_always_lock_free = 구현 정의;
    };
}
```
  
 

