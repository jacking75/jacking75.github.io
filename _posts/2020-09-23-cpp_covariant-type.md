---
layout: post
title: C++ - 공변형 반환 값과 오버라이드 룰 완화
published: true
categories: [C++]
tags: c++
---
[출처](http://cpp.aquariuscode.com/covariant-type  )  
  
C++에서는 일반적으로 반환 값만 다른 메소드를 오버라이드 할 수 없다.  
```
struct Base{
  virtual int func()=0;
};
 
struct Derived:public Base{
  float func()override; //에러! 반환 값 타입이 다르다!
};
```  
  
인수가 다르면 다른 함수로 분류되고 단순히 메소드가 증가할 뿐이지만 인수가 같은 경우는 오버라이드 대상이라서 이 코드는 컴파일 오류가 된다.  
다만 반환 값 타입이 달라도 그 형이 공변형이라면 컴파일 오류가 되지 않는다.  
  
  
요컨대 “반환 값 타입이 클래스 타입의 참조나 포인터이고 is-a 관계가 있는 것"에 관해서는 부모 클래스와 다른 형태의 반환 값이라도 오버라이드 할 수 있다는 것이다.  
```
struct Shape{};
 
//Shape과 is-a관계에 있는 Box
struct Box:public Shape{};
```  
  
```
struct Base{
  virtual Shape*get()=0;
};
 
struct Derived:public Base{
  Box* get() override;//이제는 반환 값이 형이 공변이므로 OK!
};
```
  
컴파일은 되지만 공변형을 가진 메소드는 오버라이드의 거동이 통상의 오버라이드와 다르므로 주의가 필요하다.
  
  
아래 코드를 실행하면 b->get() 에서 Box가 아닌 Shap가 표시된다  
```
#include<iostream>
 
//Shape을 정의
struct Shape {
  void print() {
    printf("shape");
  }
};

//Shape으로부터 계승한 Box를 정의
struct Box:public Shape{
  void print(){
    printf("box");
  }
};
 
 
struct Base {
  virtual~Base(){}
  virtual Shape*get()=0;
};

struct Derived:public Base {
//순수 가상 함수 get을 오버라이드 하고 Box*를 돌려준다
//기저 클래스와 함께 반환 값 형태가 다르지만 공변형이라 OK
  Box*get()override {
    return new Box();
  }
};
 
int main()
{
  Base*b=new Derived;  // Derived 생성
  auto a=b->get();  
  a->print(); 
  delete b;
  return 0;
}
```  
  
  
여기서 정말 중요한 것은 `Shape*를 반환해도 b->get()는 결코 Base::get()` 호출은 아니다 라는 것이다.  
다형성의 거동에 준하여 어디까지나 `Derived::get()`이 호출되지만, 조작하고 있는 포인터 b의 타입에 따라서 `Derived::get()`에서 지정된 타입 `Box*` 이대로 반환 값이 반환되거나 `Base::get()`에서 지정된 타입으로 암묵적으로 변환된 후 반환 되는가 라는 행동의 차이가 발생한다.  
  
익숙해지지 않으면 좀 알기 힘들다  
어쩌면 C++의 거동 속에서 톱 레벨로 알기 어렵다고도 할 수 있다.  
   
일단 공변형에 대해서 오버라이드의 룰이 완화되기 때문에 순수 가상 함수에 의한 자식 클래스로의 함수 정의 강제의 장점과 실제로 취득하는 형이 부모 클래스로 잡히지 않는 장점을 누릴 수 있다……라고 하는 것이다.  
  
자기 자신의 타입을 반환하는 함수를 정의 할 때 등 부모 클래스 고정이므로 받는 측에서 캐스트하는 것이 귀찮다.  
  
  
## 정리
1. 반환 값 타입이 다른 메소드는 오버라이드 못한다  
2. 다만 공변형의 경우에만 이 규칙이 완화된다  
3. 조작하고 있는 타입의 타입에 의해서 반환 값의 형태는 암묵적으로 변환된다  
4. 타입이 암묵적으로 변환될 뿐으로 함수의 호출 자체는 다형성  
  
  