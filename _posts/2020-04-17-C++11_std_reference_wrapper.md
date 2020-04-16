---
layout: post
title: C++11 - 컨테이너에 참조 보관하기
published: true
categories: [C++]
tags: c++ c++11 std reference_wrapper
---
`std::reference_wrapper`를 사용하면 컨테이너에 오브젝트의 참조를 보관할 수 있다.  
`std::reference_wrapper`는 `<functional>` 헤더를 사용한다.  
  
```
#include <iostream>
#include <string>
#include <vector>
#include <functional>

struct Base {
    void print() const { std::cout << "print" << std::endl; }
};

struct Integer : Base {
    int get_int() const { return 3; }
};

struct String : Base {
    std::string get_string() const { return "Hello"; }
};

int main()
{
    Integer n;
    String s;

    std::vector<std::reference_wrapper<Base> > v;
    v.push_back(std::ref<Base>(n));
    v.push_back(std::ref<Base>(s));

    
    for(Base& base : v)
    {
        base.print();
    }
    
    const int v1 = n.get_int();
    const std::string v2 = s.get_string();
}
```
  
[Wandbox](https://wandbox.org/permlink/zYqdcK6SzggM684e )  
  