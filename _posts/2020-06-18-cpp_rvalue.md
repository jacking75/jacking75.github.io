---
layout: post
title: C++ - 우측 값 참조 value category
published: true
categories: [C++]
tags: c++ rvalue c++11
---
## value category
C++의 식은 타입과는 별도로 value category라는 것을 갖는다.  
  
value category는 prvalue, xvalue, lvalue 중 하나이고, prvalue와 xvalue를 모아서 우측 값 이라고 부른다.  
  
주의해야 할 것으로  
- 같은 형을 가진 식에서도 다른 value category를 가질 있다.
- 같은 객체를 돌려주는 식이라도 다른 value category를 가질 있다.
  
  
```
string s = string("hello") + string("world");
strint t = s;
```  
위 코드에서 1 행째의 string("hello")+string("world")의 형태와  
2행째의 우변의 s 타입은 동일(string 타입)이지만, value category는 다르다(각각 prvalue와 lvalue).  
  
  
심한 예로서, 아래와 같은 경우  
```
string&& r = string("hello");
string t = r;
```  
r 타입이 string에 대한 우측 값 참조이지만  
2행째의 r 이라는 식의 value category는 lvalue 이다.  
  
  
좌측 값 식에서 무작정 우측 값을 만들고 싶은 경우는 std::move 함수를 사용하여 캐스팅한다.  
```
string s("hello");
string&& r= move(s); // move(s)는 xvalue(우측 값의 일종)
```  
  
<br>  
  
## prvalue
prvalue(pure rvalue)는 임시 객체를 생성하는 식.  
  
예:  
- 문자열 이외의 리터럴(100 이나  nullptr 이나)
- 반환 값 형이 참조 형이 아닌 함수 호출 식
- 내장 산술식, 논리식, 비교 식 등
  
<br>  
  
## xvalue
xvalue(eXpiring value)는 move 원의 객체를 나타내는 식.  
  
예:  
- 반환 값 형이 우측 값 참조인 함수 호출
- 우측 값 참조로의 캐스트 식
- xvalue인 식의 non-static 클래스 멤버에 대한 접근 식
  
<br>  
  
## lvalue  
lvalue는 임시 오브젝트가 아닌 것을 나타내는 식.  
  
예:  
- 변수 이름
- 반환 값이 좌측 값 참조인 함수 호출
- 대입식
- 문자열 리터럴  
  
<br>    
<br>  
  
## 함수의 오버로드와 우측 값
  
```
string hoge = "piyo";
string x = ???;
```  
라는 식을 생각해보자.  
`???` 부분이 hoge와 같은 좌측 값인 경우, string의 복사 생성자가 호출된다. 한편 `???` 부분이 move(hoge)와 같은 우측 값 식인 경우, string의 move 생성자가 호출된다.  
  
일반적으로 어떤 함수 f가 "좌측 값 참고를 취한 버전"과 "우측 값 참고를 취한 버전" 2가지로 오버로드 되고 있고, f 인수가 우측 값 식으로 되어 있는 경우, 우측 값 참고를 취한 버전의 f가 호출된다.  
  
복사 생성자 vs move 생성자의 예는 이 규칙의 특수 케이스이다.    
  

    