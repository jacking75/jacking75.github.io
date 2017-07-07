---
layout: post
title: C++11 - 우측값 참조. 함수의 가인수는 좌측값 참조이다
published: true
categories: [C++]
tags: c++ c++11 rvalue_reference
---
  
``` 
#include<iostream>

using namespace std;

void foo2( string& s) {
  cout << "lvalue" << endl;
};

void foo2( string&& s) {
  cout << "rvalue" << endl;
};

void foo( string& s) {
  foo2(s);
};
void foo( string&& s) {
  foo2(s);
};

int main()
{
  string s("s");

  foo(string("hoge"));   // lvalue 출력
  foo(s);    				// lvalue 출력
  foo(std::move(s));		// lvalue 출력

  return 0;
}
```
    
함수의 가인수 때는 Hoge&&를 받아도 좌측 값 참고로 된다.
  
<br> 
<br>  
  
출처: http://murasame-labo.hatenablog.com/entry/2017/03/30/201147  
