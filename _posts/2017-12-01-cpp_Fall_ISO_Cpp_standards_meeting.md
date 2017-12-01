---
layout: post
title: Trip report Fall ISO C++ standards meeting(Albuquerque) Sutter’s Mill
published: true
categories: [C++]
tags: c++ c++20 번역
---
[Trip report: Fall ISO C++ standards meeting (Albuquerque) | Sutter’s Mill](https://herbsutter.com/2017/11/11/trip-report-fall-iso-c-standards-meeting-albuquerque/)  
을 번역한 [일본 문서](https://cpplover.blogspot.kr/2017/11/c.html)를 일부 번역한 것이다.   
  
  
range-based for 에 초기화자를 쓸 수 있도록 되었다.  
```
for( auto result = f(); auto&& value : result ) ;
``` 
  
<br>  
    
<bit>에 bit_casting이 추가 되었다
```
short from = 42 ;
auto to = bit_cast<std::uint16_t>(from) ;
```
  
<br>  
      
operator <=> 가 추가 되었다.
operator <=>는 2개의 오퍼랜드의 대소 비교와 등호 비교를 한번에 할 수 있는 연산자이다.  
이 연산자를 정의해 두면 나머지 비교 연산자는 컴파일러가 자동으로 생성해 준다.  
타입 시스템에 의해 strong order, weak order, partial order 어느쪽에 대응하고 있는지도 전환된다.  
  
<br>  
      
atomic<shared_ptr<T>> 가 추가 되었다.  
shared_ptr을 atomic 하게 조작할 수 있다.  
  
<br>  
      
[[nodiscard]]가 일부 표준 라이브러리에서 사용할 수 있게 되었다
  
<br>  
      
동기(lock) 버퍼가 있는 ostream 랩퍼 라이브러리 osyncstream가 추가  
```
// 복수의 스레드에서 동시에 실행되는 함수
void f()
{
    std::osyncstream out( std::cout ) ;
    out << "hello, world!\n" ; 
}
```  
스레드 세이프한 ostream 랩퍼로 사용할 수 있다.  
  
<br>  
      
<alogorithm>와<utility>의 일부를 constexpr 대응
  
<br>  
      
<comlex>를 constexpr 대응
  
<br>  
      
atomic<floating_point_type>
  
<br>  
      
string와 string_view에 start_with/end_with 추가
  
<br>  
      
현재 책정 중의 constexpr 대응 new, vector, string은 이후 몇 차례 회의에서 드래프트에 들어갈 전망이라고 할 정도로 본격 논의되고 있다   
  