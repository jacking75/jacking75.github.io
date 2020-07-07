---
layout: post
title: C++ - 함수의 디폴트 인자 값을 동적으로 바꾸기
published: true
categories: [C++]
tags: c++ 함수
---
일반적인 함수의 기본 인자 값  
```
void TEST_1(int nValue = 5 )
{
    std::cout << __FUNCTION__ << ". " << nValue << std::endl;
}
```  
  
위의 디폴트 값은 고정인데 아래처럼 동적으로 바꿀 수 있다.
```
#include <iostream>
#include <time.h>

int GetNumber()
{
    return (int)time(NULL);
}
 
void TEST_2(int nValue = GetNumber() )
{
    std::cout << __FUNCTION__ << ". " << nValue << std::endl;
}


int main()
{
    TEST_2();
    
    return 0;
}
```
  
  
```
#include <iostream>
#include <time.h>

void TEST_1(int nValue = 5 )
{
    std::cout << __FUNCTION__ << ". " << nValue << std::endl;
}

int GetNumber()
{
    return (int)time(NULL);
}

void TEST_2(int nValue = GetNumber() )
{
    std::cout << __FUNCTION__ << ". " << nValue << std::endl;
}

int g_nValue = 0;
void TEST_3( int nValue = ( 
                g_nValue = GetNumber(),
                g_nValue+1) )
{
    std::cout << __FUNCTION__ << ". " << nValue << std::endl;
}

int main()
{
    TEST_1();

    TEST_2();

    _sleep(1000);
    TEST_2();

    TEST_3();

    return 0;
}
```  