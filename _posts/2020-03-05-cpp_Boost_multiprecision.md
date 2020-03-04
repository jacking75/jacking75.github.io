---
layout: post
title: C++ - boost::multiprecision::cpp_int
published: true
categories: [C++]
tags: c++ boost
---
[출처](https://qiita.com/Lily0727K/items/ab089a4974c614e7f861 )   
  
다배수 길이 정수를 다룰 때는 boost::multiprecision::cpp_int를 사용한다.  
  
```
#include <bits/stdc++.h>
#include <boost/multiprecision/cpp_int.hpp>

using namespace std;
using boost::multiprecision::cpp_int;

int main() {
  cpp_int a, b;
  cin >> a >> b;
  cout << (a > b ? "GREATER" : (a == b ? "EQUAL" : "LESS")) << endl;
}
```
  
