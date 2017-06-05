---
layout: post
title: Win32API - CRITICAL_SECTION의 OwningThread
published: true
categories: [C++]
tags: c++ c memory new
---
Windows Vista부터 CRITICAL_SECTION의 내부 구조가 바뀌었다고 한다.   
이 중 눈여겨 볼 것은 CRITICAL_SECTION의 멤버 변수 중 OwningThread 인데 이 변수에는 CRITICAL_SECTION을 소유중인 스레드의 ThreadID가 설정된다.  
그래서 멀티스레드에서 CRITICAL_SECTION를 사용할 때 특정 스레드가 락을 획득하지 못하고 대기중이라면 어떤 스레드에서 락을 소유중인지를 OwningThread를 통해서 알 수 있다.  
  
![](/images/2017_criticalsection_owningThread.PNG)   
  
<br> 


