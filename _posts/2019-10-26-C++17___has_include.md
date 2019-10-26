---
layout: post
title: C++17 - __has_include
published: true
categories: [C++]
tags: c++ c++17 __has_include
---
포함(include)하는 파일이 존재하는지 확인하는 프리프로세서 기능이다.  
  
아래는 optional 파일이 존재하는지 확인한다.  
  
```
#if __has_include(<optional>)
    #include <optional>
#endif
```
혹은  
```
#if __has_include("optional")
    #include "optional"
#endif
```   
      