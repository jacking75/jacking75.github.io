---
layout: post
title: C++17 - false sharing와 true sharing 제어
published: true
categories: [C++]
tags: c++ c++17 sharing thread cache
---
[출처](https://faithandbrave.hateblo.jp/entry/2016/07/08/174657 )  
병행 프로그래밍으로 문제가 될 수 있는 캐시 무효화 문제를 제어 할 수 있다.  
  
  
## false sharing 제어
  
```
struct keep_apart {
    atomic<int> cat;
    atomic<int> dog;
};
```
  
이런 구조가 있으면 cat와 dog가 같은 캐시 라인에 탈 수 있다. 스레드 1은 cat 변수, 스레드 2는 dog 변수를 조작하는 상황에서 각각 공통의 캐시를 무효화 해 버려서 성능 저하 문제가 발생할 수 있다.  
이러한 상황을 **false sharing** 라고 하는데,이것을 방지하기 위해 <new> 헤더에 `std::hardware_destructive_interference_size` 라는 상수가 정의 되어 있다.  
  
이 정수를 구조체 멤버 정렬로 지정하여 각각의 멤버가 독립적인 캐시 라인을 타서 false sharing을 피할 수 있다.  
```
struct keep_apart {
    alignas(hardware_destructive_interference_size) atomic<int> cat;
    alignas(hardware_destructive_interference_size) atomic<int> dog;
};
```
   
   
   
## true sharing 제어
**true sharing** 라는 용어는 일반적이지 않은듯 하지만 의도적으로 여러 멤버를 동일한 캐시 라인에 실을 때 이렇게 부른다.  
  
이런 일을 하기 위해 <new> 헤더에 `std::hardware_constructive_interference_size` 라는 상수가 정의 되어 있다.   
구조체 전체에 이 상수를 정렬 지정하여 전체 구조체가 동일한 캐시 라인에 타는 것으로 예상 할 수 있다.  
  
```
struct alignas(hardware_constructive_interference_size) together {
    atomic<int> dog;
    int puppy;
};
```
  
  
  