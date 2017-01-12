---
layout: post
title: C++11 - 최근접 짝수 반올림 std::nearbyint
published: true
categories: [C++]
tags: c++ c++11 std::nearbyint
---
### 개요

실수를 정수로 변환할 때 ceil()로 소수점 올림하거나, floor()로 소수점 버림을 한다.  
  
nearbyint는 정수로 변환할 실수의 가장 가까운 짝수로 올림 혹은 버림을 한다.
이러한 방식을 "bankers’rounding" 이라고 부른다
  
<br> 
<br>  

 
### 문법

cmath 헤더파일을 포함해야 한다.


<br> 
<br>  


### 사용 예

```
#include <iostream>
#include <cmath>

int main()
{
    std::cout << std::ceil(0.3) << std::endl; // 1 출력
    std::cout << std::floor(0.5) << std::endl; // 0 출력
    
    std::cout << std::nearbyint(0.4) << std::endl; // 0 출력
    std::cout << std::nearbyint(0.5) << std::endl; // 0 출력
    std::cout << std::nearbyint(1.5) << std::endl; // 2 출력
    std::cout << std::nearbyint(2.5) << std::endl; // 2 출력
    std::cout << std::nearbyint(3.5) << std::endl; // 4 출력
}
```




