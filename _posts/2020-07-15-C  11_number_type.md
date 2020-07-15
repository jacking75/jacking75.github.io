---
layout: post
title: C++ - C++11에서 추가된 사이즈 엄밀한 정수 타입
published: true
categories: [C++]
tags: c++ c++11 int8_t int16_t int32_t int64_t
---
[출처](http://qiita.com/YukiMiyatake/items/361259344def99a439d5  )  
  
C++11에서 새로 추가됨  
```
int8_t, int16_t, int32_t, int64_t, uint8_t, uint16_t, uint32_t, uint64_t, int_fast8_t, int_fast16_t, int_fast32_t, int_fast64_t, uint_fast8_t, uint_fast16_t, uint_fast32_t, uint_fast64_t, int_least8_t, int_least16_t, int_least32_t, int_least64_t, uint_least8_t, uint_least16_t, uint_least32_t, uint_least64_t, intmax_t, uintmax_t, intptr_t, uintptr_t
```
  
  
## [u]intX_t
int32_t 나 가장 많이 사용되지만 무려 optional 이므로 처리 시스템에 의해서는 미  구현일 수도 있으므로 이 경우는 아래의
`[u]int_fastX_t` 나 `[u]int_leastX_t` 를 쓰게 된다.  
예를 들면 1바이트가 9비트 처리계에서는 4Byte=36bit 이므로 int32_t는 정의되지 않을 것이다.  
대신 `int36_t`가 정의될 수도...  
이 모델은 엄밀한 비트 수로 패딩 없는 크기로 확보한다.  
```
sizeof(int8_t) = 1
sizeof(int16_t) = 2
sizeof(int32_t) = 4
sizeof(int64_t) = 8
```
  
  
## [u]int_fastX_t
가장 실행 속도가 빠르게 되는 바이트 수가 확보된다(처리계 의존).  
즉 패딩이 된다.  
```
sizeof(int_fast8_t) = 1
sizeof(int_fast16_t) = 8
sizeof(int_fast32_t) = 8
sizeof(int_fast64_t) = 8
```
  
  
## [u]int_leastX_t
저장 가능한 가장 작은 사이즈가 확보된다(처리계 의존).  
즉 패딩 없음.  
```
sizeof(int_least8_t) = 1
sizeof(int_least16_t) = 2
sizeof(int_least32_t) = 4
sizeof(int_least64_t) = 8
```  
  
  
## [u]intmax_t
최대 크기의 정수형이 확보된다  
```
sizeof(intmax_t)=8
```
  
  
## [u]intptr_t
포인터 크기가 확보된다.  
```
sizeof(intptr_t) = 8
```
  