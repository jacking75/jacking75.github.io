---
layout: post
title: C++17 - shared_ptr 배열에 대응
published: true
categories: [C++]
tags: c++ c++17 shared_ptr array
---
C++17에서는 아래처럼 사용할 수 있다.  
  
```
std::shared_ptr<double[1024]> p1 {new double[1024]};
std::shared_ptr<double[]> p2 {new double[n]};

double* p = p1[0];
```
  
  