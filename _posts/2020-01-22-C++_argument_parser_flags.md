---
layout: post
title: C++ - flags. 프로그램 실행 시의 인자를 파싱하는 라이브러리
published: true
categories: [C++]
tags: c++ flags argument 라이브러리
---
- C++17로 표준으로 구현.
- 헤더 파일 1개로 구성.
- 저장소: https://github.com/sailormoon/flags
- 라이센스: 퍼블릭도메인(완전 마음대로 사용 가능)
  
  
  
## 예제
  
```
#include "flags.h"
 
void ParseConfig(int argc, char* argv[])
{
    const flags::args args(argc, argv);
 
    const auto ip = args.get<std::string>("ip");
    if (!ip) {
        std::cerr << "No IP. :(" << std::endl;
        return;
    }
    std::cout << "IP: " << *ip << std::endl;
 
 
    const auto port = args.get<int>("port");
    if (!port) {
        std::cerr << "No Port. :(" << std::endl;
        return;
    }
    std::cout << "Port: " << *port << std::endl;
 
}
 
int main(int argc , char* argv[])
{  
    ParseConfig(argc, argv);
    return 0;
}
```
  
실행  
<pre>
app.exe --ip="127.0.01" --port=32452
</pre>
  