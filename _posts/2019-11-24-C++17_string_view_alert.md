---
layout: post
title: C++17 - string_view의 주의점
published: true
categories: [C++]
tags: c++ c++17 string_view
---
[출처](http://d.hatena.ne.jp/yohhoy/20180810/p1 )  
  
- string_view 클라스 == 문자열에 대한 읽기 전용 뷰.
    - std::string와 같이 null 종단 문자열 이외도 올바르게 다룰 수 있다. null 종단 문자를 도중에 포함 할 수 있다.
- string_view 클래스는 참조처 문자열의 소유권을 관리하지 않는다.
    - 옛스러운 raw pointer 와 비슷한 dangling 뷰에 아주 주의해야 한다.
- 함수 파라메타 타입에 있어서는 string_view 이용은 안전.
    - null 종단 문자열(const char*) 혹은 문자열(const std::string&)로의 뷰를 통일적, 효율적으로 다룰 수 있다.
    - 함수 구현에서 소유권이 필요로 된다면 std::string에 저장(＝문자열 실체를 복사) 한다.
    - std::string과는 다르고, string_view::data 멤버 함수는 null 종단 문자열을 보증하지 않기 때문에 null 종단 문자열을 요구하는 레거시 API 호출 용도로는 적합하지 않다.
- 함수 반환 값 타입이나 클래스 데이터 멤버로서 string_view 이용에는 충분히 유의한다. dangling 뷰 위험이 있다.
    - 참조처 오브젝트의 생존 기간(lifetime)을 문서화 한다.
- 임시 문자열 오브젝트를 가리키는 string_view 변수를 명시적으로 만들지 않는다.
    - 지역 변수에는 string_view 타입이 아닌 auto&& 이용을 검토해야 한다.
  
```
// 문자열 타입 string을 반환하는 함수
std::string f();

std::string_view s1 = f();
// NG: 함수 반환 값(임시 문자열 오브젝트)는 파괴 되기 때문
//     이 시점에서 s1은 덩글링 뷰가 된다

auto&& s2 = f();
// OK: 함수 반환 값(임시 문자열 오브젝트) 우측 값 참조 타입으로 넘기면
//     이 생존 기간은 변수 s2의 스코프가 끝날 때까지 연장 된다.
```  
  
