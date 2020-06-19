---
layout: post
title: C++17 - map과 unordered_map의  try_emplace()와 insert_or_assign()
published: true
categories: [C++]
tags: c++ c++17 map unordered_map
---
try_emplace() 멤버 함수: 삽입 실패 시에 주어진 파라미터 args...를 변경하지 않는다.  
insert_or_assign() 멤버 함수: 삽입에 실패하면 덮어 쓴다.  
  
  
```
std::map<std::string, std::unique_ptr<Foo>> m;
m["foo"] = nullptr;

std::unique_ptr<Foo> p(new Foo);
auto res = m.try_emplace("foo", std::move(p));
assert(p); // p는 유효
```
  
```
std::map<std::string, int> m;

auto [it1, b1] = m.insert_or_assign("foo", 42);
std::cout << '(' << it1->first << ", " << it1->second << "), " << std::boolalpha << b1 << '\n';

auto [it2, b2] = m.insert_or_assign("foo", 0);
std::cout << '(' << it2->first << ", " << it2->second << "), " << std::boolalpha << b2 << '\n';
```
  
  