---
layout: post
title: C++11 - std::get_time
published: true
categories: [C++]
tags: c++ c++11 std::get_time
---
### 개요

**>> get_time(tmb, fmt)** 식을 사용하여 현재 입력 스트림에서 전달 받은 로컬 시간  값을 변환 형식 문자열에 맞추어 tm 오브젝트로 변환한다.
  
<br> 
<br>  

 
### 문법

```
template< class CharT >
/*unspecified*/ get_time( std::tm* tmb, const CharT* fmt);
```

- tmb는 std::tm 타입 오브젝트를 가리키는 포인터로 여기에 결과 값이 저장된다.
- fmt는 변환 형식을 지정하는 null로 끝나는 문자열을 가리키는 포인터
    - 형식은 여기를 참조 http://en.cppreference.com/w/cpp/io/manip/get_time  

<br> 
<br>  


### 사용 예

```
#include <iostream>
#include <sstream>
#include <iomanip>
#include <ctime>
 
int main()
{
  std::istringstream s("20170110");
 
  std::tm x = {};
  s >> std::get_time(&x, "%Y%m%d");
 
  char buf[255];
  strftime(buf, sizeof(buf), "%Y-%m-%d", &x);
  std::cout << buf << std::endl;
}
```

<pre> 결과
2017-01-10
</pre>
  
  
<br> 
<br>  


### 참고
- http://dev.activebasic.com/egtra/2017/01/10/945/
- http://en.cppreference.com/w/cpp/io/manip/get_time

