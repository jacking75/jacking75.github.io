---
layout: post
title: C++11 - 고정형 정수 타입
published: true
categories: [C++]
tags: c++ c++11 int32_t
---
C++11에서 고정형 길이의 정수 타입이 새로 생겼음.  
고정형 길이라는 것은 int8_t, int16_t 등 타입에 숫자가 붙는 것을 말함.   
플랫폼에 상관 없이 크기가 고정 되어 있어서 크로스 플랫폼 개발에 사용하면 좋음.  
  
  
  
## 헤더 파일
  
```
#include <cstdint>
```
  
  
## 정수 타입
int8_t, int16_t, int32_t, int64_t
  
int_fast8_t, int_fast16_t, int_fast32_t, int_fast64_t  
  
int_least8_t, int_least16_t, int_least32_t, int_least64_t
  
uint8_t, uint16_t, uint32_t, uint64_t
  
uint_fast8_t, uint_fast16_t, uint_fast32_t, uint_fast64_t
  
uint_least8_t, uint_least16_t, uint_least32_t, uint_least64_t
  
  
## 매크로 정수
INT8_MIN, INT16_MIN, INT32_MIN, INT64_MIN
  
INT_FAST8_MIN, INT_FAST16_MIN, INT_FAST32_MIN, INT_FAST64_MIN
  
INT_LEAST8_MIN, INT_LEAST16_MIN, INT_LEAST32_MIN, INT_LEAST64_MIN
  
INT8_MAX, INT16_MAX, INT32_MAX, INT64_MAX
  
INT_FAST8_MAX, INT_FAST16_MAX, INT_FAST32_MAX, INT_FAST64_MAX
   
INT_LEAST8_MAX, INT_LEAST16_MAX, INT_LEAST32_MAX, INT_LEAST64_MAX
  
UINT8_MAX, UINT16_MAX, UINT32_MAX, UINT64_MAX
   
UINT_FAST8_MAX, UINT_FAST16_MAX, UINT_FAST32_MAX, UINT_FAST64_MAX
  
UINT_LEAST8_MAX, UINT_LEAST16_MAX, UINT_LEAST32_MAX, UINT_LEAST64_MAX
   
 
 
  
더 자세한 설명은 아래 링크로   
출처: https://en.cppreference.com/w/cpp/types/integer    