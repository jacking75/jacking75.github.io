---
layout: post
title: C++ 컴파일 시 조건에 의해 변수 선언 타입을 변경하기
published: true
categories: [C++]
tags: c++ compile std::conditional
---
컴파일 시에 정해진 조건 C의 bool 값에 의해 타입 A 혹은 B를 선택하고 이 타입의 변수를 선언하고 싶은 경우 의사 코드는 아래와 같다.  

```
if (C)
{
  A x;
}
else
{
  B x;
}
```  

이것을 구현하기 위해 std::conditional을 사용하여  

```
typename std::conditional<C, A, B>::type x
```  
로 한다.  
  
  
### 이용 예
난수 생성 엔진으로 32비트판의 std::mt19937와 64비트판의 std::mt19937_64이 준비 되어 있다.  
템플릿에서 uint32_t가 지정되거나 uint64_t를 지정하는 것에 의해 이것들을 같은 변수 이름으로 선언하고 싶다.  
이런 때 std::is_same을 사용하여   
```
typename std::conditional<std::is_same<INT, uint32_t>::value, std::mt19937, std::mt19937_64>::type m;
```  
로 쓸 수 있다. typename은 컴파일러에게 이것이 타입이라고 알려준다.  
  
아래처럼 랩핑해서 사용할 수 있다.  
  
test.cpp  
```
#include <iostream>
#include <random>

template <class INT>
struct Hoge {
  typename std::conditional<std::is_same<INT, uint32_t>::value, std::mt19937, std::mt19937_64>::type m;
  INT operator()() {
    return m();
  }
};

int main() {
  Hoge<uint32_t> h32; // 32비트 난수
  Hoge<uint64_t> h64; // 64비트 난수 
  for (int i = 0; i < 10; i++) {
    std::cout << h32() << " " << h64() << std::endl;
  }
}
```  

<pre>
$ g++ test.cpp
$ ./a.out
3499211612 14514284786278117030
581869302 4620546740167642908
3890346734 13109570281517897720
3586334585 17462938647148434322
545404204 355488278567739596
4161255391 7469126240319926998
3922919429 4635995468481642529
949333985 418970542659199878
2715962298 9604170989252516556
1323567403 6358044926049913402
</pre>
  
     
<br>
<br>  

출처: https://qiita.com/kaityo256/items/15932998ee78c1577725 