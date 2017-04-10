---
layout: post
title: C++14 - shared_(timed_)mutex가 지원하는 Reader 스레드 수
published: true
categories: [C++]
tags: c++ c++14 mutex shared_mutex
---
C++14 표준 라이브러리의 shared_timed_mutex 클래스는 적어도 10000 스레드 이상의 Reader 스레드에서의 공유 lock(shared lock) 동시 획득을 지원한다.  
또 동시 획득 가능한 공유 lock 수 상한을 넘은 경우도 공유 lock을 획득 할 수 있을 때까지 Reader 스레드가 대기하는 것을 보증한다.  
  
 
<br>  
<br>  


출처: http://d.hatena.ne.jp/yohhoy/20170213/p1

