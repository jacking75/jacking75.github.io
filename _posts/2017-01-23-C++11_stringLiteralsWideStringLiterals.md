---
layout: post
title: C++11 - 문자열 리터럴과 wide 문자열 리터럴 결합
published: true
categories: [C++]
tags: c++ c++11 std::string std::wstring
---
C99 호환을 위해서 만들어진 기능으로 문자열 리터럴과 wide 문자열 리터럴을 결합하면 wide 문자열로 결합한다.
문자열 리터럴인란 멀티바이트 문자열을 뜻하고, wide 문자열 리터럴은 유니코드 문자열을 뜻한다.  
C++11 이전에는 문자열 리터럴과 wide 문자열 리터럴 결합에 대해서 미정의 동작이었다.
  
<br> 
<br>  

 
### 예제

```
#include <iostream>

int main()
{
  const wchar_t s1[] = "hello" L" world";
  const wchar_t s2[] = L"hello" " world";

  std::wcout << s1 << std::endl;
  std::wcout << s2 << std::endl;
}
```



### 참고
- https://cpprefjp.github.io/lang/cpp11/string_literal_concatenation.html

