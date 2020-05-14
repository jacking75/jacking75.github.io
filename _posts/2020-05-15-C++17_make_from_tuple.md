---
layout: post
title: C++17 - 튜플을 유저 정의 타입으로 변환하는 make_from_tuple 함수
published: true
categories: [C++]
tags: c++ c++17 tuple make_from_tuple
---
예제 코드  
```
#include <iostream>
#include <string>
#include <tuple>

struct Person {
    int id;
    double bodyHeight;
    std::string name;

    Person(int id, double bodyHeight, const std::string& name)
        : id(id), bodyHeight(bodyHeight), name(name) {}
};

int main()
{
    std::tuple<int, double, std::string> t(1, 152.3, "Alice");
    Person p = std::make_from_tuple<Person>(t);

    std::cout << p.id << std::endl;
    std::cout << p.bodyHeight << std::endl;
    std::cout << p.name << std::endl;
}
```
  