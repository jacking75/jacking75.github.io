---
layout: post
title: C++17 - static_assert의 에러 메시지 생략 가능
published: true
categories: [C++]
tags: c++ c++17 static_assert
---
static_assert는 C++11에서 생긴 기능으로 컴파일 타임에 식의 true, false 여부를 조사해서 false인 경우 지정한 에러 메시지를 출력한다.  
```
static_assert(a == b, "a와b는 같아야 한다");
```  
    
C++17에서는 static_assert 에 에러 메시지 지정을 하지 않아도 괜찮다.  
```
static_assert(a == b); 
```  
식이 false 일 때 어떤 메시지를 출력할지는 규정이 정해지지 않아서 각 컴파일 마다 다르게 출력될 수 있다.  
  
<br>  
    
예제 코드   
```
#include <iostream>


int main()
{
    const int a = 10;
    const int b = 11;
    
    // C++11
    static_assert(a == b, "a와b는 같아야 한다");
    
    // C++17
    static_assert(a == b);                     
    
    return 0;
}
```   
  
[Wandbox](https://wandbox.org/permlink/TRUWro7tm8q2QU5w)  
    