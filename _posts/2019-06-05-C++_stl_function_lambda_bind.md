---
layout: post
title: C++ - STL function, lambda, bind 사용 예
published: true
categories: [C++]
tags: c++ 
---
  
```
/*
 * std_function1.cpp
 * Copyright (C) 2014 kaoru <kaoru@bsd>
 */

#include <iostream>
#include <functional>
using namespace std;

struct Foo {
        Foo(const int n) : i_(n) {}
        void print_add(const int n) const { std::cout << i_ + n<< std::endl; }
        int i_;
};

struct PrintFunctor {
        // 인수를 표시만 하는 함수오브젝트
        void operator()(int i) {
                std::cout << i << std::endl;
        }
};

void
print_number(const int i)
{
        std::cout << i << std::endl;
}

int main(int argc, char const* argv[])
{
        // 보통의 함수
        std::function< void(int) > f_func = print_number;
        f_func(3);

        // 람다식
        std::function< void(int) > f_lambda = [=](int i) { print_number(i); };
        f_lambda(6);

        // 바인드
        std::function< void() > f_bind = std::bind(print_number, 9);
        f_bind();

        // 클래스의 멤버 함수
        std::function< void(const Foo&, int) > f_member = &Foo::print_add;
        Foo foo (1);
        f_member(foo, 3);       // 1+3 = 4

        // 함수 오브젝트
        std::function< void(int) > f_func_obj = PrintFunctor();
        f_func_obj(11);

        return 0;
}
```  
  
