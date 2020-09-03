---
layout: post
title: C++ - f()와 f(void)의 차이
published: true
categories: [C++]
tags: c++ f(void)
---
VC++을 통해서 멤버 함수를 정의할 때 파리미터가 없는 함수의 경우 파라미터 리스트가 들어가는 자리에 void가 들어가는 것을 볼 수 있다.  
  
기본적으로 `void f()`와 `void f(void)`의 함수 시그네쳐는 같은 것이다.  
다만 파라미터 리스트에 `void`를 사용하는 것은 ‘빈 파라미터 리스트’라는 것을 뜻하기 때문에  
```
void f( int, void*); // OK
```  
는 문제가 없지만
```
void f( int, void); // ERROR
```
  
는 에러가 된다.  
  
  