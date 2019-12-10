---
layout: post
title: C++17 - shared_ptr의 참조 카운트와 data reace
published: true
categories: [C++]
tags: c++ shared_ptr c++17
---
C++17 표준 라이브러리에서는 스마트 포인터인 std::shared_ptr의 unique 멤버 함수는 비추천이 되었다.  
상위 호환이 되는 use_count 멤버 함수는 잔존하지만 스레드 간 동기에는 관여하지 않는다는 요건이 명확화 되어서 멀티스레드 실행에서는 반환 값은 근사(approximate)가 된다는 취지의 Note가 추가된다.  
    
- shared_ptr 참조 카운트를 이용한 스레드 간 동기는 하지 않는다. 데이터 경합(data race)의 리스크가 있다.  
- shared_ptr<T>::unique() 멤버 함수를 이용하지 않는다. C++17에서 비추천(deprecated).  
- shared_ptr::use_count() 멤버 함수는 디버깅 용도로만 이용한다. 멀티스레드 처리에서는 신뢰할 수 없는 값을 반환할 가능성이 있다.
     
