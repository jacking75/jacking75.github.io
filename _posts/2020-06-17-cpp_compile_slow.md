---
layout: post
title: C++ - C 언어 보다 왜 컴파일이 느릴까?
published: true
categories: [C++]
tags: c++ compile
---
## 일반적으로 C++로 작성된 코드가 C 보다 컴파일 시간이 느린 이유 
그 이유는 주로 아래의 5개이다.    
- C++ 문법의 복잡성, 소스 코드의 parse(어휘 분석, 구문 분석, 의미 분석)에 시간을 요하고 있다
- 템플릿의 실체화에 시간이 걸린다
- 복잡한 최적화를 실시 할 필요가 있다
- 표준 라이브러리의 기능이 너무 많아서 포함하면 코드의 양이 폭발적으로 증가
- 헤더에 구현이 적혀 있는 경우가 많다
  
  
  
## 컴파일 시간을 줄이는 방법
컴파일 시간을 줄이는 방법은 여러 가지 있지만, 역시 **헤더 include를 쓰지 않는 것** 이 가장 중요하다.  
특히 표준 라이브러리는 parsing에 시간이 걸리기 때문에 비교적 코드가 분할 되어 있는 boost를 사용하거나, algorithm 등이 필요하지 않으면 스스로 구현하는 것이 좋다.  
  
또는 원래 STL 와의 인터페이스를 구현하지 않는 것도 하나의 방법이라고 생각한다.  
예를 들어, 지정된 경로가 디렉토리를 가리키는지 여부를 나타내는 함수 is_dir을 구현할려고 할 때  
```
bool is_dir(const std::string& s);
```
  
  
위 코드를 쓰는 것만으로 <string> 헤더를 include 하는 것은 낭비가 크므로 대신에 아래의 2개의 인터페이스를 준비한다  
```
inline bool is_dir(const char* str) { is_dir(str, strlen(str)); }

bool is_dir(const char* str, std::size_t length);
```
  
  
참고: **STL의 include 문제는 C++20의 모듈** 을 사용하면 거의 대부분 해결할 수 있다.    