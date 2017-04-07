---
layout: post
title: C++14 - recursive_(timed_)mutex의 재귀 lock 상한 수
published: true
categories: [C++]
tags: c++ c++14 mutex recursive_mutex
---
C++11 표준 라이브러리의 recursive_mutex, recursive_timed_mutex 클래스는 동일 스레드에서의 재귀 lock 획득 횟수의 상한은 미정의로(unspecified) 되어 있다.  
단 상한 회수를 넘는 try_lock 조작은 실패하고, lock 조작은 예외를 던지는 것을 보증 한다.  
    
  
<br>  
<br>  


출처: http://d.hatena.ne.jp/yohhoy/20170214/p1

