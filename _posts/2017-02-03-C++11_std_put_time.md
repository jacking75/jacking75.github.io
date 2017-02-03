---
layout: post
title: C++11 - std::put_time
published: true
categories: [C++]
tags: c++ c++11 std::put_time
---
### 개요

날짜 서식을 출력한다.
  
<br> 
<br>  

 
### 문법

```
template <class CharT>
unspecified put_time(const struct tm* tmb, const CharT* fmt);
```

- tmb 은 유효한 tm 타입 오브젝트를 가리키는 포인터
- fmt 는 유효한 문자 배열을 가리키는 포인터

<br> 
<br>  


### 사용 예

```
#include <iostream>
#include <chrono>
#include <ctime>
#include <iomanip>

using std::chrono::system_clock;

int main() 
{
  system_clock::time_point p = system_clock::now();

  std::time_t t = system_clock::to_time_t(p);
  const tm* lt = std::localtime(&t);
  std::cout << std::put_time(lt, "%c") << std::endl;
}
```

```
#include <iostream>
#include <iomanip>
#include <ctime>
 
int main()
{
    std::time_t t = std::time(nullptr);
    std::tm tm = *std::localtime(&t);
    
    std::cout.imbue(std::locale("ko_KR.utf8"));
    std::cout << "ko_KR: " << std::put_time(&tm, "%c %Z") << '\n';
}
```


<br> 
<br>  


### 참고
- https://cpprefjp.github.io/reference/iomanip/put_time.html
- http://en.cppreference.com/w/cpp/io/manip/put_time

