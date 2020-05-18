---
layout: post
title: C++11 - map에 삽입 insert, emplace
published: true
categories: [C++]
tags: c++ c++11 map container emplace stl
---
아래 방식은 unordered_map에도 적용된다.  
  
## map::insert
insert를 사용하면 key와 value의 pair를 삽입할 수 있다.  
```
map<int,int> dic{};

dic.insert(std::make_pair(1,3));
```
  
  
## map::emplace
insert와 비슷하지만 이쪽은 값을 key와 value의 생성자에 전송한 값을 구축한다.  
```
std::map<int,int> dic{};

dic.emplace(1,3); // dic.insert(std::make_pair(1,3))와 같다
```
   
귀찮게 pair를 만들지 않아서 좋다.  
  
  