---
layout: post
title: 멤버 변수에 대한 포인터를 이용하여 타입의 특성을 판정
published: true
categories: [C++]
tags: c++ template meta
---
## 멤버 변수에 대한 포인터라는 것은
우선, 예로서 단순히 정수를 래핑 한 클래스를 보자

```
class int_wrapper {
public:
  int data;
  int_wrapper(int data) : data(data) {}
};
```
  
위 코드에서 int_wrapper::data로의 포인터 p는

```
int int_wrapper::*p = &int_wrapper::data;
```
  
라고 쓸 수 있다. p에 넣는 것은 int_wrapper 클래스 내의 int 타입의 멤버 변수에 대한 주소이다.  
객체가 아니라 클래스로 쓰고 있는 것으로 알겠지만 p가 가리키는 것은 메모리 상의 구체적인 곳이 아니라 상대적인 것이다.  
실질적으로 오프셋이 이라고 생각해도 좋다.  
  
사용할 때는 아래와 같다  
  
```
#include <iostream>

int main(void) {
  int_wrapper foo(3);
  int int_wrapper::*p=&int_wrapper::data;
  std::cout << foo.*p << std::endl;
}
```
  
덧붙여서, 멤버 변수뿐 아니라 멤버 함수도 같은 기법으로 다룰 수 있다. (C++에 익숙한 사람이라면 멤버 변수에 대한 포인터와 멤버 함수에 대한 포인터라는 것은 약간 다른 체계라는 것을 쉽게 알 수 있으므로 여기에서는 언급하지 않는다.)  
  
  
## 존재하지 않는 타입으로의 포인터  
멤버 변수에 대한 포인터에 대해서는 대상이 되는 클래스에 해당하는 타입의 멤버가 존재하지 않아도 정의할 수 있다.  
가령 위의 예에서는 int_wrapper에는 double 타입의 멤버 변수는 없지만  
  
```
double int_wrapper::*q;
```
  
라는 식으로 변수 q를 만들어도 문제 없다. 물론 해당하는 멤버 변수가 존재하지 않으므로 캐스트 하지 않고 변수 q에 넣는 것은 널 포인터만 될뿐이다.  
  
  
## 존재하지 않는 타입을 언제 사용하나?
존재하지 않는 타입을 가리키는 포인터는 문법의 일관성 때문에 우연히 생긴 것이라고 생각하지만 템플릿에 제약을 붙이는 용도로 쓸 수 있다.  
  
아래의 템플릿이 템플릿 인수로서 클래스 밖에 전달할 수 없다.  
  
```
template<class T>
class class_wrapper {
private:
  typedef int T::*dummy;
public:
  T data;
  class_wrapper(T obj) : data(obj) {}
};
```  
  
클래스 T가 int 타입의 구성원을 가지든 말든 int T::* 타입이 존재하는 것은 허용되지만 T가 클래스가 아니면 멤버를 갖지 못하므로 에러가 된다.  
  
테스트 해 보자.  
  
```
int main(void) {
  int_wrapper foo(3);
  class_wrapper<int_wrapper> bar(foo);  // int_wrapper는 클래스이므로 OK
  class_wrapper<int> baz(5);            // int 는 클래스가 아니므로 NG
  return 0;
}
```
  
  
C++11 이후부터는 type_traits에 있는 is_class를 사용하면 이런 제약을 표현할 수 있습니다.  
is_class 쪽이 무엇을 표현하는지 이름부터 알기 쉬우므로 더 좋은 방법이라고 할 수 있다.
  
  
## 제약의 중요성
"움직이지 않았으면 하는 경우"를 상정하는 것은 중요하다. 특히 범용적인 라이브러리는 만든 사람이 생각했던 것과는 전혀 다른 용도로 사용될 지도 모른다.  또 규모가 어느 정도 커지면 자신마저 그다지 신용할 수 없게 된다.  
예상 밖의 용도로 사용될 때 컴파일 오류가 되는 것은 버그를 막기 위해 아주 유용하다.
  
     
<br>
<br>  

출처: http://qiita.com/SaitoAtsushi/items/e2847efef72d7e7b9959 