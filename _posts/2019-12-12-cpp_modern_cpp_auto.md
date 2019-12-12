---
layout: post
title: C++ - 4개의 auto, 4개의 C++ 규격
published: true
categories: [C++]
tags: c++ auto C++20
---
[원문](https://qiita.com/yohhoy/items/b28258884312c4f55599   )  
  
2020년에 출시될 예정인 「C++20」에서는 아래와 같은 함수 템플릿을 쓸 수 있다.  
  
cpp20.cpp  
```
template <auto N>
auto func(auto x)
{
  auto r = N + x;
  return r;
}

// 호출 예
assert( func<1>(0.5) == 1.5 );
```
  
이 글에서는 C++ 언어 사양 변경에 의해 도입된 각 auto 키워드의 역할을 설명한다.  
  
  
## C++20 시대
C++20 에서는 함수 파라메터 타입에 `auto`를 쓸 수 있다.   
이것은 C++ 언어 사양의 Concept 도입과 함께 추가된 것이다.  
함수 템프릿 정의 단축 기법이다.   
  
cpp20.cpp  
```
template <auto N>
auto func(/*👉*/auto/*👈*/ x)
{
  auto r = N + x;
  return r;
}
```
  
  
  
## C++17 시대
C++17에서는 템플릿 파라메터 타입에 auto를 쓸 수 있도록 되었다.  
즉 비 타입 템플릿 파라메터의 auto 선언이다.  
  
cpp17.cpp  
```
template </*👉*/auto/*👈*/ N, typename U>
auto func(U x)
{
  auto r = N + x;
  return r;
}
```  
  
  
  
## C++14 시대
C++14에서는 함수 반환 값 타입에만 auto를 쓸 수 있다.   
함수 본체에 있는 return 문의 내용에서 반환 값 타입이 추론 되고, 보통 함수의 반환 값 추론이라는 기능이다.  
  
cpp14.cpp  
```
template <typename T, T N, typename U>
/*👉*/auto/*👈*/ func(U x)
{
  auto r = N + x;
  return r;
}
```
  
  
  
## C++11 시대
C++11은 키워드 auto가 타입 추론 의미를 가지도록 된 최초의 버전이다.  
엄밀하게는 변수의 타입 추론과 반환 값의 타입을 뒤에 두는 함수 선언구문의 auto 키워드는 다른 의미를 가지지만 큰 차이는 없다.  (전자= 타입 추론 / 후자=플레이스 홀더).  
  
cpp11.cpp  
```
template <typename T, T N, typename U>
auto func(U x) -> decltype(N + x)
{
  /*👉*/auto/*👈*/ r = N + x;
  return r;
}

// 호출 예
assert( func<decltype(1), 1>(0.5) == 1.5 );
```
  
  
