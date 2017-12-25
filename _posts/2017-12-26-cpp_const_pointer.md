---
layout: post
title: C++ - 포인터 변수에서 const 위치에 따른 차이 
published: true
categories: [C++]
tags: c++ const pointer
---
변수에 const를 사용하는 이유와 사용 방법은 아주 쉽다.  
그런데 포인터 타입 변수에 const를 붙였을 때 포인터의 앞이나 뒤 어디에 붙여야 할지 헷갈릴 때가 있다. 
  
포인터에 const를 붙이는 경우 아래에 따라서 차이가 발생한다.  
- * 앞에 const를 붙인 경우는 포인터를 가리키는 곳이 불변이 된다  
```
int value = 42;

const int* p = &value;
p = &value;     //OK
*p = 10;         //NG
  
- * 후에 const를 붙인 경우는 포인터 변수 자체가 불변이 된다
```
int value = 42;

int* const p2 = &value;
p2 = &value;      //NG
*p2 = 10;          //OK
```
  
<br>  
  
참고로 cosnt int * const 식으로 양쪽에 const를 붙인 경우는 『 변수 』도 『 실체 』도 모두 불변이 된다.  
```
int value = 42;

const int* const p = &value;
p = &value;           //NG
*p = 10;               //NG
```
  
<br>  
<br>  
    
[참고](http://qiita.com/pink_bangbi/items/a36617bf1d5923743d69)