---
layout: post
title: C++ - Redis 사용 방법 정리 GitHub 저장소
published: true
categories: [C++]
tags: c++ redis github
---  
GitHub에 C++에서 Redis 클라이언트 라이브러리에 대해서 간단하게 정리하였다.  
[jacking75/examples_cpp_redis](https://github.com/jacking75/examples_cpp_redis)  
  
만약 Redis를 key-value 정도로 간단하게 사용한다면 **RedisCpp-hiredis** 와 **redis_client** 를 추천한다.  
hiredis 라이브러리만 있다면 위 라이브러리는 헤더파일 1개만으로 redis를 사용할 수 있어서 준비가 간단하다  
RedisCpp-hiredis 보다는 redis_client가 기능이 더 많다.  
둘 다 Windows에서 사용 및 1-헤더 파일로 사용할 수 있도록 수정한 것이다.  
  
Redis의 최신 기능까지 사용하고 싶다면 **r3c**를 추천한다.  
  
위에서 열거한 라이브러리 이외에도 더 있으니 원하는 것을 찾기 바란다.    
  
<br/>  
    

[jacking75/examples_cpp_redis](https://github.com/jacking75/examples_cpp_redis )  저장소가 도움이 되었다면 스타 부탁한다~  
  