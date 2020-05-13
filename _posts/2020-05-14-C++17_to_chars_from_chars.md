---
layout: post
title: C++17 - 숫자를 문자열로 변환하는 to_chars(), 문자열에서 숫자로 변환는 from_chars()
published: true
categories: [C++]
tags: c++ c++17 to_chars from_chars
---
  
- 로케일은 C 로케일(POSIX 로케일) 고정
- 함수 내에서 동적 메모리 할당이 없다. 즉 호출 측에서 만들어줘야 한다
- 포맷은 파라메터로 주고, 자동적으로 포맷을 해석할 수 없다
- 사용할 수 있는 포맷은 최소한
    - + 부호의 지정 불가
    - 문자열 중 공백 불가
    - # 에 의한 소수점 이하 자릿수 지정 불가
    - 소문자만 허용하고, 대문자는 사용 불가
    - 16 진수에 0x를 붙일 수 없다
- 예외 던지기는 없다
  
  
예제 코드  
```
#include <iostream>
#include <utility>
#include <limits>

int main()
{
    {
        char str[std::numeric_limits<int>::digits10 + 2] = {0}; // '-' + NULL
        int input = 123456;

        // 정수 input을 10진수로 문자열로 변환하고 출력
        std::to_chars(std::begin(str), std::end(str), input, 10);
        std::cout << str << std::endl; // 123456
    }
    {
        const char input[] = "-123456";
        int value = 0;

        // 10진수 정수가 들어간 문자열을 int로 변환
        std::from_chars(std::begin(input), std::end(input), value);
        std::cout << value << std::endl; // -123456
    }
}
```



  
  