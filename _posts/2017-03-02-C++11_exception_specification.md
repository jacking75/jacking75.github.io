---
layout: post
title: C++11 - 예외 지정
published: true
categories: [C++]
tags: c++ c++11 exception
---
C ++에서는 함수에 예외 지정이라는 것을 작성할 수 있다.   
이것은 C++98에서 부터 있는 기능으로 throw (T1, T2, ...)라는 문법으로 함수가 밖으로 던지는 예외를 지정하는 기능이다.
  
```
// C++98/03
void f() throw( int, double );
```
  
  
만약 함수가 예외 지정에 지정한 것 이외의 형태를 던진 경우 std :: unexpected가 호출된다.
  
```
void f() throw( int ) 
{
    throw 0 ; // OK
    throw 0.0 ; // 실행 시 에러. std::unexpected() 
}
``` 
  
  
예외 지정은 함수에서 던진 예외를 지정하는 기능으로 함수의 외부에 예외를 던지지 않으면 문제가 없다.
  
```
void f() throw( int ) 
{
    try {
        throw 0.0 ; // OK
    } catch( ... ) { } // OK 예외를 밖으로 던지지 않는다
}
```  
  
  
위의 이야기는 C++11 이전 이야기다.
    
C++11 에서는 **예외 보증  없음** 지정 기능만 특화한 noexcept가 추가되었다.  
noexcept를 지정하면 함수는 외부에 예외를 던지지 않는다라고 지정하게 된다.
  
```
void f() noexcept;
```
  
  
만약 외부에 예외를 던진다면 std :: terminate가 호출된다.  
또한 noexcept에는 bool 형식의 컴파일시 상수 피 연산자를 가지는 문법이 있다.  
식이 true 일 때는 예외 지정 없음을 나타내고, false 때는 예외 지정 없음이 아니라는 의미를 갖는다.
  
```
void f() noexcept(true) ; // 예외를 밖으로 던지지 않는다
void g() noexcept(false) ; // 예외를 밖으로 던질 가능성이 있다
```
  
  
이렇게 하면 컴파일시 메타 프로그래밍에서 함수의 예외 지정 없음 여부를 전환 할 수 있게 된다.  
  
```
void f1() ;  // 예외를 밖으로 던질 수 있다
void f2() throw() ; // 예외를 밖으로 던지지 않는다
void f3() noexcept ; // 예외를 밖으로 던지지 않는다
void f4() noexcept( true ) ; // 예외를 밖으로 던지지 않는다
void f5() noexcept( false ) ; // 예외를 밖으로 던진다
```
  
  
또한 C ++ 11에서는 noexcept 식 이라는 것이 추가 되었다.  
이것은 피연산자 식이 예외를 던질 여부를 bool로 반환한다. 피연산자 식은 미 평가 식이다.
  
```
void f() ;
void g() noexcept ;

int main()
{
    noexcept( f() ) ; // false
    noexcept( g() ) ; // true
}
```  
  
  
<br>  
<br>  

출처: https://cpplover.blogspot.kr/2014/10/c.html

