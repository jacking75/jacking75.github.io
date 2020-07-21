---
layout: post
title: C++ - 생성자에 용도 별로 이름 붙이기
published: true
categories: [C++]
tags: c++ class tag
---
최근 C++ 표준 라이브러리와 Boost의 작은 유행으로 태그 디스패치의 태그를 사용자에게 명시적으로 지정하게 함으로써 함수 오버로드를 손쉽게 하는 방법이 몇몇 곳에서 나타나고 있다.  
  
```
// Boost.Container의 vector의 예
// 태그 정의
struct default_init_t {};
constexpr default_init_t default_init {};

// vector 생성자
vector::vector(size_type, default_init_t);

// 3요소의 배열을 준비하고, 각 요소를 미 초기화 상태로 한다
vector<T> v(3, default_init);
```
  
이 경우 단지 3 이라는 첫번째 인수를 전달 하고 값 초기화한 리사이즈 조작이 되겠지만 default_init 라는 태그를 붙임으로써 "미 초기화 상태로 한다" 라는 프로그래머 의도를 부가하고 있다.  
  
통상의 함수의 경우 별명의 함수를 준비해서 용도를 나누지만 생성자는 이름이 없어서 이런 태그 붙임이 이루어지고 있다.  
C++ 표준 라이브러리에도 여러 태그를 사용자가 지정하는 경우가 있다.    
  
```
mutex m;

void f()
{
    m.lock();

    // 이미 락을 건 mutex를 받는다.
    lock_guard<mutex> lk(m, adopt_lock);
    
}
```  
  
```
namespace std {
  struct adopt_lock_t { };
  constexpr adopt_lock_t adopt_lock { };
}
```  
  
이러한 태그 붙임은 생성자 오버로드에 의미를 갖게 하는 목적으로 사용하기 때문에 편리하고 사용하기 좋은 기법이다.    
  