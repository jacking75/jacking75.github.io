---
layout: post
title: C++ - new로 메모리 할당 실패 시 예외 발생하지 않게 하기
published: true
categories: [C++]
tags: c++ new nothrow new(nothrow)
---
아래 코드를 실행하면 아마 메모리를 할당할 수 없어서 예외가 발생할 것 이다.  
```
long long n = 1LL << 63LL;
int *p = new int[n];
```  
  
그러나 예외가 아닌 null pointer을 발생하고 싶다면 아래와 같이 한다.  
```
long long n = 1LL << 63LL;
int *p = new(nothrow) int[n];
if (p == nullptr) cout << "Null Pointer" << endl;
```
  