---
layout: post
title: C++ - 맴버 함수 포인터와 배열을 사용하여 멤버 함수를 번호로 지정하여 호출 하는 방법
published: true
categories: [C++]
tags: c++ pointer class
---
[출처](http://qiita.com/shiro_naga/items/5967f6cd1710e7b78677  )  
  
```
#include <iostream>
#include <memory>

using ub4 = unsigned int;

// 외부에 공개해서 이용하는 클래스
class A {
public: // 클래스 밖에서 using을 사용하고 싶으므로 public 로 한다.
    class impl; 
private:
    std::unique_ptr<impl> pim;
public:
    A();
    ~A();
public:
    void func_call( ub4 fn_num );
};

// 멤버 함수 포인터로 타입을 선언
using a_fn_ptr_t = void ( A::impl:: * )();



class A::impl {
    static constexpr ub4 pf_num = 4; // 정수로 배열 요소 수를 지정
    static a_fn_ptr_t func_ar[ pf_num ];
public:
    impl();
    ~impl();
public:
    void func_call( ub4 fn_num );
private:
    void func_00();
    void func_01();
    void func_02();
    void func_03();
};


// 배열에 멤버 함수 포인터를 넣는다
a_fn_ptr_t A::impl::func_ar[] = {
    &A::impl::func_00,
    &A::impl::func_01,
    &A::impl::func_02,
    &A::impl::func_03
};

A::A() :
    pim ( new impl() )
{}

A::~A()
{}

void A::func_call( ub4 fn_num )
{
    pim->func_call( fn_num );
}

A::impl::impl()  { printf("A::impl::impl() \n");}
A::impl::~impl() { printf("A::impl::~impl()\n");}


void A::impl::func_call( ub4 fn_num )
{
    if( fn_num >= pf_num ) return; 
    ( this->*( func_ar[ fn_num ] ) )(); // 배열에 들어가 있는 멤버 함수를 호출한다.
}


void A::impl::func_00() { printf("A::impl::func_00()\n"); }
void A::impl::func_01() { printf("A::impl::func_01()\n"); }
void A::impl::func_02() { printf("A::impl::func_02()\n"); }
void A::impl::func_03() { printf("A::impl::func_03()\n"); }


int main()
{
    A a;
    a.func_call( 0 );
    a.func_call( 1 );
    a.func_call( 2 );
    a.func_call( 3 );
    a.func_call( 4 );
    a.func_call( 5 );
    a.func_call( 6 );

    return 0;
}
```
  
