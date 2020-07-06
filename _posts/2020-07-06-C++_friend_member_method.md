---
layout: post
title: C++ - 다른 클래스의 특정 멤버 함수만을 friend로 선언
published: true
categories: [C++]
tags: c++ friend
---
보통 다른 클래스를 friend로 선언할 때 그 클래스 전체를 선언하는 경우가 많은데 클래스 전체가 아닌 일부 멤버 함수만을 friend로 선언할 수 있다.  
```
struct TEST2;

struct TEST1
{
    void T1();
    void T2();
};
 
struct TEST2
{
    friend void TEST1::T1();

private:
    int m_nValue;
};

void TEST1::T1()
{
    TEST2 t2;
    t2.m_nValue = 1;
}

void TEST1::T2()
{
    // 아래 주석을 제거하면 컴파일 에러
    //TEST2 t2;
    //t2.m_nValue = 1;
}
```
  
위 코드는 TEST1의 T1()은 TEST2의 모든 멤버에 접근할 수 있지만 T2()는 TEST2의 public 멤버에만 접근할 수 있다.  
  
