---
layout: post
title: C++ - C++ 표준 위원회 문서 P0563R0:Vector Front Operations (pdf)
published: true
categories: [C++]
tags: c++ c++20 번역 vector 
---
흥미로운 글이 있어서 일부 번역했습니다.(오랜전에)    
(원본 https://cpplover.blogspot.kr/2017/02/c-p0550r0-p0601r0.html )  
    
[PDF P0563R0:Vector Front Operations](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0563r0.pdf)  
  
vector에 push_front와 pop_front가 없는 이유는 O(N)의 비효율적인 조작 때문이다.  
vector에 대해 선두에서의 insert, erase는 나머지 요소를 모두 1개씩 당기거나 밀어내야한다.  
  
그런데 최근의 하드웨어의 사정에 의해 완전히 바뀌어 버렸다.  
이제는 외관상 순서보다 데이터의 국소성이 중요하게 되었다.  
데이터를 인접하는 메모리로 바꾸는 조작보다 국소성 있는 메모리 확보 쪽이 훨씬 비용이 더 비싼 처리가 되었다.  
  
실제로 마이크로 벤치 마크에서도 선두로의 insert, erase는 vector가 list, deque 보다 빠르다.  
그래서 더는 vector에 push_front와 pop_front을 쓰지 못할 이유는 없다.  
참고로, vector에 push_front와 pop_front를 붙이면 std::queue의 내부 컨테이너로 사용할 수 있다.  
  