---
layout: post
title: C++ - _pFirstBlock == pHead 에러
published: true
categories: [C++]
tags: c++ vc _pFirstBlock
---
실행 프로그램에서 공용 dll을 사용하는 경우 아래와 같은 에러가 발생할 때가 있다.  
  
<pre>
File: f:\dd\vctools\crt_bld\self_64_amd64\crt\src\dbgheap.c
Line: 1424
 
Expression: _pFirstBlock == pHead
</pre>  
  
  
- 2중으로 free에서도 발생하는 에러로 dll 내부에서 free 한 것을 그 뒤에 다시 free 하면 이런 에러가 발생한다.
- dll 에서 할당한 메모리를 exe 측에서 free 하는 경우. 이 문제 해결 방법은 2가지
    - static lib으로 바꾼다.
    - 런타임을 Multi-threaded Debug DLL (/MDd)로 바꾼다.
  