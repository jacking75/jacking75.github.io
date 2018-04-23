---
layout: post
title: RPS(Receive Packet Steering)와 RFS(Receive Flow Steering)  
published: true
categories: [Network]
tags: rps rfs
---
### RPS(Receive Packet Steering)
RPS는 수신한 패킷을 처리하는 네트워크 부하를 다중 코어로 분배할 수 있다.  
이 기술은 TCP/IP 같은 프로토콜을 패킷으로서 병행적으로 처리할 수 있다.  
매핑은 해시 테이블 메커니즘에서 행해진다.  
수신한 큐에는 CPU에 대한 마스크 인덱스로 특정의 해시 패킷 헤더가 이용된다.  
RPS는 어떤 CPU가 특정 네트워크 큐를 실행할지를 결정한다.  
  
### RFS(Receive Flow Steering) 
RFS는 RPS를 발전시킨 기술이다.  
RPS에서는 CPU 선택이 랜덤으로 하지만 선택된 CPU 코어가 다른 태스크에서 오버로드 되어 있는 경우에는 문제가 생긴다.  
RFS에서는 "recvmsg()" 라는 시스템 콜을 내고 있는 CPU을 선택함으로써 cache 이용 효율을 높인다.  
  
### 멀티 코어 CPU 시스템 개발에 반영
RPS와 RFS는 Linux kernel 2.6.35 이후의 시스템에 매우 효과적이며, 멀티 코어 CPU 시스템 개발에 있어서는 이들을 잘 사용할 필요가 있다.  
  
### RFS와 RPS의 장점
- 네트워크 성능을 3배, 시스템 전체의 이용 효율을 2배로 해 준다.
- cache의 효과적인 이용
- 프로토콜을 병행적으로 처리
- 대기 시간이 짧다
- 싱글 큐와 멀티 큐 지원
- Linux에서 구현 되어 있으므로 이용하기 쉽다　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　