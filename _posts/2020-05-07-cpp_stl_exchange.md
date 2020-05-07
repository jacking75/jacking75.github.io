---
layout: post
title: C++ - std::exchange
published: true
categories: [C++]
tags: c++ stl exchange c++14
---
C++14부터 사용 가능.  
  
아래의 파일을 포함해야 한다  
```
#include <utility>
```
  
함수 원형  
```
template< class T, class U = T >
T exchange( T& obj, U&& new_value );
```
  
<br>  
  
예제 코드  
  
```
flags_type flags(flags_type newf)
{ return std::exchange(flags_, newf); }

s.flags(12)
```
  
  
```
void f() { std::cout << "f()"; }

void (*fun)();
std::exchange(fun,f);
```
  
  
```
std::vector<int> v;
std::exchange(v, {1,2,3,4});
```  
  