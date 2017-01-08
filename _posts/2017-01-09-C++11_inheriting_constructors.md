---
layout: post
title: C++11 - 생성자 상속(inheriting constructors)
published: true
categories: [C++]
tags: c++ c++11
---
### 개요

부모 클래스에서 정의한 생성자들을 자식 클래스에서 그대로 사용할 수 있는 편의 기능이다.

  
<br> 
<br>  
  
  
### 문법

자식 클래스에서 using 키워드를 사용하여 부모 클래스 이름과 부모 클래스의 생성자 이름을 **::** 구분으로 호출한다.  
자식 클래스는 복수의 부모 클래스를 상속한 경우에도 각 부모 클래스의 생성자를 사용할 수 있다.
  
```
class Base { };

class Derived : public Base {
  
  // Base 생성자를 Derived에서 암묵적으로 선언한다
  using Base::Base;
};
```  
  
<br>  
<br>  

### 사용 예

```
#include <iostream>
#include <string>

class Base {
public:
  Base() {}
  Base(int) { std::cout << "Base(int)" << std::endl; }
  Base(const std::string&) { std::cout << "Base(const std::string&)" << std::endl; }  
};

class Derived : public Base {
  // Base 생성자를 Derived에서 암묵적으로 선언한다
  using Base::Base;
};

int main()
{
  Derived d1(3);       // OK
  Derived d2("hello"); // OK
}
```




