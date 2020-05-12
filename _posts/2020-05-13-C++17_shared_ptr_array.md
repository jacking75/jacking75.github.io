---
layout: post
title: C++17 - shared_ptr에서 배열 사용
published: true
categories: [C++]
tags: c++ c++17 shared_ptr
---
예전에는 배열을 사용하는 경우 메모리 해제 방식을 직접 지정해야했다.  
[shared_ptr에서 배열(array) 해제하는 방법](https://psychoria.tistory.com/268)  
  
C++17에서는 아래처럼 간단해졌다.  
  
```
std::shared_ptr<double[1024]> p1 {new double[1024]};
std::shared_ptr<double[]> p2 {new double[n]};

double* p = p1[0];
```
  
  