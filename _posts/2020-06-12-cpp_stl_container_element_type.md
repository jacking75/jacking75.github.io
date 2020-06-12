---
layout: post
title: C++ - STL 컨테이너의 인스턴스에서 요소의 형을 얻는다
published: true
categories: [C++]
tags: c++ stl
---
  
```
template<class T>
struct foo{
 T operator()( ... ){ ... };
};

std::array< double, ... > a = {{ ... }};
bar( foo<  >() ); // <  > 중에 a의 요소 형 double을 넣고 싶다..

```  
  
```
bar( foo< decltype(a)::value_type >() );
// decltype(a)::value_type은 double

```  
  