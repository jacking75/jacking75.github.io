---
layout: post
title: C++ - 추상 기저 클래스
published: true
categories: [C++]
tags: c++ virtual
---
아래 코드처럼 virtual 이 붙은 가상 기반 클래스는 "모호성" 을 피하기 위함이다.  
  
```
class B : virtual  public A{}
class C : virtual public A{}
class D : public B, public C{}
```
  
가상 기반 클래스가 아니라면 D 클래스가 생성시에 A 를 상속 받을 때,  
B에서 받을지? 아니면 C에서 받을지 모호하다. 컴파일러는 애매모호한 것을 아주 싫어한다.  
  
그러므로 virtual 키워드를 사용하여 D 클래스에서는 B 와 C 클래스가 상속받은 A 클래스를 하나로 인지하도록 하여, 모호성을 없애주고 D는 오직 하나의 A 클래스만을 인식한다.  
  