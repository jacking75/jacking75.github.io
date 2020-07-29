---
layout: post
title: C++ - ref-qualifier 유무로 오버로드 할 수는 없다
published: true
categories: [C++]
tags: c++ overload ref c++11
---
[출처](http://cpplover.blogspot.kr/2010/09/ref-qualifier.html )  
  
C++11에는 implicit object parameter(`*this`를 참조하는 객체)를 lvalue 참조로 받거나 rvalue 참조로 받는지를 지정하는 기능이 있다. 이를 참고 참고 수식자(ref-qualifier)라고 한다.  
```
struct S
{
    // void S::f(void) &
    void f() & ;
    // void S::f(void) &&
    void f() && ;

    // 참조 수식자가 생략된 함수
    // this의 참조처는 lvalue 참조
    void g() ;
} ;

int main()
{
    S s ;
    s.f() ; // void S::f(void) &를 호출
    std::move(s).f() ; // void S::f(void) &&를 호출
}
``` 
  
  
참조 수식자 유무는 함수의 타입에 영향을 미친다.   
여기에서는 void f() &;와, void g() ;는 의미적으로는 같지만 다른 타입이다.   
다만 참조 수식자의 유무에 따른 오버로드를 할 수 없다.    
```
struct S
{
    // void S::f(void) &
    void f() & ; 
    // void S::f(void) &&
    void f() && ;

    // 에러
    void f() ;
};
```  
  
만약 어떤 함수 이름에 참조 수식자를 쓸 경우 그 함수 이름의 오버로드 함수에는 모든 참조 수식자가 씌어 있어야 한다.  
어느 오버로드 함수에서 참조 수식자를 생략한다면, 어느 오버로드 함수에도 참조 수식 인자를 쓰면 안 된다.  
  