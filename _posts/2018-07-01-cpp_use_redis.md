---
layout: post
title: C++ - redis 사용하기
published: true
categories: [C++]
tags: c++ redis
---  
게임 서버를 개발할 때 사용하는 기술이 모바일 시대 이전에 비해서 많이 다양해졌는데 거의 대부분 꼭 사용하고 있는 것이 **redis** 이다.  
다른 언어들은 redis 사용이 간단한데 C++은 간단하지는 않아서 정리를 해 봤다.  
boost를 사용하면 아주 쉽게 사용할 수 있고, boost를 사용하지 않거나 VC 지원을 해야하는 경우 좀 더 복잡해진다.  
  
<br>    
    
## 공식 redis C++ 클라이언트 라이브러리 정리

- redis 공식 사이트에 가면 각 언어 별 라이브러리가 있다.
    - C++ http://redis.io/clients#c--
- [hiredis 사용하기](http://qiita.com/Ki4mTaria/items/d73cf3d244c903d493eb)
- [Windows용 redis](https://github.com/MSOpenTech/redis)
  
<br>    
  
  
### redis_client
- https://github.com/shawn246/redis_client
- Windows 아직 지원하지 않음.
    - 만약 사용하고 싶다면 read-write lock인 'pthread_rwlock_t'를 윈도우 버전에 맡게 수정해야 한다.
- 한 쌍의 .h/.cpp 로 구성.
- hiredis에 의존한다. 즉 꼭 필요하다.
- pipeline 지원.
- cluster까지 지원.
- connection pool 지원.
- Thread sage.
- Reconnect automatically
- 현재 구현 예정인 것
    - Optimize the reconnect function.
    - Support pub/sub and transaction.
    - Support scan in an unsafe way.    
  
<br>    
  
  
### cpp_redis
- https://github.com/cylix/cpp_redis
- C++ 11 필요.
- Windows 지원.
- pipeline, pub/sub 지원.
- (2016.10)현재 VS로는 DLL 만드는 솔루션만 제공(이것도 CMake 사용)
    - static lib으로 사용하고 싶다면 따로 솔루션 파일을 만들어야 한다.
  
<br>    
  
### async-redis
- https://github.com/hamidr/async-redis
- An async redis library with sentinel support based on (libevpp|ASIO)  
  
  
<br>    
  
  
### xRedis
- https://github.com/0xsky/xredis
- Windows 지원.
    - 바로는 사용 못하고 좀 작업을 해야함.
- hiredis 필요.
- support Redis master slave connection, Support read/write separation
- connection pool
- multi thread safety
- 주요 문서가 중국어
  
<br>    
  
  
### redisclient
- https://github.com/nekipelov/redisclient
- Boost.asio(header only) based Redis-client library.
- C++ 11 필요.
- Windows 지원.
  
<br>    
  
  
### redisclient
- Boost.asio based Redis-client library.
- https://github.com/nekipelov/redisclient 
  
<br>    
  
  
### poco 공식 redis
- https://github.com/pocoproject/poco
- poco 라이브러리 공식 redis 라이브러리.
- 1.80 버전에 정식 지원 예정. 현재는 github에서 poco를 입수하면 redis 라이브러리가 포함되어 있음.
- Windows 지원.
  
<br>    
  
  
### redis-client
- https://github.com/RedisCppTeam/redis-client
- Windows 지원
- poco 라이브러리 필요
  
<br>    
  
  
### redis3m
- https://github.com/luca3m/redis3m
- Windows 지원.
- hiredis and boost libraries 필요.
- 1년 정도 방치된 상태임. 2016/10 기준.
  
<br>    
  
  
### redox
- https://github.com/hmartiro/redox
- C++11 지원 필요.
- linux만 지원.
- hiredis and libev 필요.
- Modern, asynchronous, and wicked fast C++11 client for Redis
  
<br>    
  
  
### r3c
- https://github.com/eyjian/r3c
- Windows 비 지원.
- C++03 지원.
- hiredis 필요.
- .h/.cpp 두 개의 파일로 구성.
- 문서가 중국어로 되어 있음.
  
<br>    
  
  
## 비 추천
- [redis-cplusplus-client](https://github.com/mrpi/redis-cplusplus-client)
    - 너무 오래된 프로젝트로 방치된 상태임. 2016/10 기준으로 6년 전 프로젝트.
