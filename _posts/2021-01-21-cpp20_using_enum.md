---
layout: post
title: C++20 - Using enum
published: true
categories: [C++]
tags: c++ c++20 using enum
---
using enum은 C++20의 새로운 기능.  
scoped enum을 namespace의 using directive를 사용할 수 있다.  
  
C++17까지는 enum을 아래처럼 사용한다.  
```
enum struct color { red, green, blue } ;

void f( color c )
{
    switch( c )
    {
        case color::red :
            break ;
        case color::green :
            break ;
        case color::blue :
            break ;
    }
}
```
  
  
using enum을 사용하면 아래처럼 할 수 있다.  
```
enum struct color { red, green, blue } ;

void f( color c )
{
    using enum color ;
    switch( c )
    {
        case red :
            break ;
        case green :
            break ;
        case blue :
            break ;
    }
}
```
  
scoped enum은 암묵적인 타입 변환을 할 수 없으므로 기존의 enum 사용에 변환가 없지만 사용을 좀 더 편하게 하기 위해 namespace의 using 디렉티브와 같은 기능인 using enum이 추가 되었다.  
  

      