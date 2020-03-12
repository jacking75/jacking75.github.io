---
layout: post
title: C++ 오픈소스 서버 라이브러리
published: true
categories: [Network]
tags: network server opensource cpp
---
## libuv_cpp11
- 저장소: https://github.com/wlgq2/libuv_cpp11
- libuv를 Modern C++로 랩핑한 라이브러리
  
  
## uvw
- 저장소: https://github.com/skypjack/uvw
- libuv를 Modern C++로 랩핑한 라이브러리
- TODO
    - VC 프로젝트 만들기
    - libuv와 연결 시켜 줘야 한다
  
  
## znet  
- 저장소 url: https://github.com/starwing/znet
- 개발에 사용해도 좋을 듯.
- TCP, UDP
- IOCP, epoll 사용.
- lua를 지원한다.
- C/C++ 언어로 만들어져 있다.
- 네트워크 기능은 znet.h, znet.hpp에 모두 구현 되어 있다. znet만 C++로 만들어져 있다
- 코드 분석 난이도(1 제일 쉬움, 10 제일 어려움): 6
    - C++ 학습용으로 좋을 듯  
  
  
## zsummerX
- 저장소 utl: https://github.com/zsummer/zsummerX
- IOCP/EPOLL/SELECT
- C++ 언어로 만들어져 있다  
  
    
## brynet
- https://github.com/IronsDu/brynet 
- IOCP, epoll
- 분석하기.
- 불 필요한 기능 제거하기.
- 성능 및 안정성 테스트.    
  
  
  
## uWebSockets
- 저장소 url: https://github.com/uNetworking/uSockets
    - uWebSockets https://github.com/uNetworking/uWebSockets
- Linux만 지원한다.
    
  
## handy
- 저장소: https://github.com/yedf/handy
- Linux만 지원한다.
  
  
## cloudwu/socket-server
- 저장소 url: https://github.com/cloudwu/socket-server
- Linux만 지원한다.
- 중국의 MMORPG '몽환서유' 개발자인 cloudwu씨가 만든 서버 코드
- tcp/udp
- epool과 kqueue 사용.
- C 언어로 만듬.
  
### 파일 구성
- 간단하게 구성되어 있다.
- 5개의 파일로 이중 하나는 테스트 코드 파일이다.
| 파일            | 설명 |
|-----------------|------|
| socket_poll.h   | epoll과 kqueue 사용할 선택하기 위한 헤더 파일  |
| socket_epoll.h  | socket의 epoll 정의. 레벨 트리거 사용   |
| socket_kqueue.h | socket의 kqueue 정의. bsd 계열에서 사용  |
| socket_server.h | 서버 네트워크 함수와 구조체 선언  |
| socket_server.c | 서버 네트워크 함수와 구조체 정의   |
| test.c          | socket_server을 사용한 테스트 프로그램   |
  
  
  
## Skynet2
- 저장소 url: https://github.com/cloudwu/skynet2
- 중국의 MMORPG '몽환서유' 개발자인 cloudwu씨가 만든 서버 코드로 '몽환서유'의 기본 코드의 버전 2.
    - 넷이즈에서 아직도 사용 중이라고 함.
- TCP
- epoll
- C로 만들어져 있다.
- [Skynet 설계 설명](https://coolspeed.wordpress.com/2015/12/20/skynet_design_philosophy/ )
- https://github.com/cloudwu/skynet  
- https://github.com/cloudwu/skynet_sample 
  
  
  
## fflib_net
- https://github.com/fanchy/fflib 에서 네트워크 부분만 분리.
    - 이 프로젝트는 rpc 통신을 목적으로 하고 있음.
    
  
  
## jubatus-mpio
- 저장소 https://github.com/jubatus/jubatus-mpio
- 관련 글 http://frsyuki.hatenablog.com/entry/20100412/p1 
  