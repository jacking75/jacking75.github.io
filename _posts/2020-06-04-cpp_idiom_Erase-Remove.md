---
layout: post
title: C++ - idiom - Erase-Remove
published: true
categories: [C++]
tags: c++ idiom
---
[원문](https://qiita.com/MoriokaReimen/items/cf492903ac93c613a3e1)  
  
  
아래는 STL의 컨테이너에서 특정 요소를 삭제하는 코드.
```
std::vector<int> a;

/*a에 요소를 추가하는 코드는 생략*/

std::remove(a.begin(), a.end(), 100);
```  
  
이것으로 사라진 것이 아니다. 실은 remove 함수는 특정 요소를 컨테이너의 뒤에 만든 후 선두의 특정 요소에 대한 반복문을 반환하기만 한다.  
정말로 지우려면 이 Erase-Remove를 사용하자.  
  
```  
erase-remove.cpp
std::vector<int> a;

/*a에 요소를 추가하는 코드는 생략*/

a.erase(std::remove(a.begin(), a.end(), 100), v.end());
```  
  