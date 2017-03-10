---
layout: post
title: tm 에서 time_t로 UTC로 변환 하는 함수
published: true
categories: [C++]
tags: c++ tm time_t
---
  
| x         | time_t→ tm 구조체 | tm 구조체 → time_t |
| ----------- |:------------------:| -------------------:|
| 현지 시각 | localtime        | mktime            |
| UTC       | gmtime           | ?                 |


표의 "?" 부분은 UTC로 tm 구조체에서 time_t로 변환하는 함수가 C 및 C++ 표준 라이브러리에 없다.  

어쩔 수 없이 비표준이라도 좋으니 없는지 찾았보았다.  
Unix 계열의 일부에는 timegm, Visual C++에는 _mkgmtime 라는 함수가 나타났다.  

- Linux glibc, BSD계열, Cygwin 등
    - timegm 함수(Linux, FreeBSD)
- Visual C++
    - _mkgmtime

이 밖에 Linux의 man에는 Unix계에서 일반적으로 통용될 듯한 방법이 설명되어 있다.
setenv에서 환경 변수 TZ을 설정하고 mktime을 호출하는 방식이다.  

또  std::chrono::system_clock::time_point ← → std::time_t ← → std::tm ← → (std::put_time, std::get_time)이라는 std::chrono::system_clock::time_point에서의 입출력 과정에서 필요로 한다.



<br>  
<br>  


출처: http://dev.activebasic.com/egtra/2017/01/03/941/
