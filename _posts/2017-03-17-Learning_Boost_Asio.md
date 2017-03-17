---
layout: post
title: Boost.Asio 공부하기
published: true
categories: [C++]
tags: network c++ boost asio
---
나는 2007년부터 Boost.Asio를 온라인 게임 서버 개발에 처음 사용하였다(업무에 사용하는 것을 말한다.그전에 공부는 했었다)  
  
당시 주위의 게임 개발자를 통해서 생각외로 사용하는 곳이 꽤 있었다.  
이 후 한국 및 해외의 유명 회사들도 C++ 네트워크 프로그래밍에 사용하고 있다고 들었다. 
  
한국의 온라인 게임이 PC에서 모바일로 넘어가면서, 중국 게임 퍼블리셔의 영향력이 쎄지면서 온라인 게임 서버 OS 플랫폼이 거의 Windows 독점에서 Linux로 넘어가면서 Boost.Asio를 사용하는 곳이 더 많아 진 것 같다. 

Linux에서 온라인 게임서버 개발 경험이 없고, 크로스 플랫폼 지원을 쉽게 하기 위해서 Boost.Asio를 많이 사용하는 것 같다.  

나는 온라인 게임서버용 네트워크 엔진을 완전 처음부터 만드는 것 보다 Boost.Asio를 기반으로 만드는 것을 추천한다. 빠른 개발 속도와 안정성, 성능을 같이 가져 갈 수 있다.  
그러나 공짜가 없듯이 어느 정도 네트워크 프로그래밍 실력이 있어야 하지 완전 0 에서 시작하면 쉽지 않다.  
  
네트워크 프로그래밍 경험이 별로 없지만 빨리, 성능 좋은 서버를 만든다면 상용 네트워크 엔진 사용을 추천한다.


### [간단] 초보자가 Boost.Asio 공부하기
- IOCP 공부하기  
Boost.Asio를 서버에서 사용한다면 거의 대부분 비동기 I/O로 사용할 것이다. 그래서 비동기 I/O에 대한 지식이 필요하다. 책이나 네이버 검색을 통해 먼저 IOCP에 대해서 꼭 공부하기 바란다.  

- 네이버에서 Boost.Asio를 검색해본다.  
Boost.Asio가 신기술이 아니고 10년이 넘은 기술이라서 한글로 된 문서가 웹에 있다.  
한글로 된 문서를 보고 싶다면 구글링 보다 네이버 검색이 더 좋다. 물론 영어 잘 알면 구글링이 더 좋다 ^^

- [KGC 2012] Boost.asio를 이용한 네트웍 프로그래밍  
2012년에 KGC에서 내가 강연한 것이다. 

- [e-book] Boost.Asio를 이용한 네트워크 프로그래밍(한빛미디어)  
[구입하기](http://www.hanbit.co.kr/store/books/look.php?p_code=E7889843127)  
활용에 초점을 둔 책이다. 저자가 나이다 ^^  
초보자 용으로 동기 IO, 비동기 IO, 채팅 서버 만들기가 들어가 있다.

- [e-book] Boost.Asio C++ Network Programming Cookbook(Packt)  
[구입하기](https://www.packtpub.com/application-development/boostasio-c-network-programming-cookbook)  
내가 보기에 위의 내 책보다 설명이 좀 더 자세한 것 같다. 그래서 영어 잘 한다면 내 책보다 이 책을 추천한다.  
단 영어가 약하다면 내 책이나 네이버에서 검색한 Boost.Asio 관련 글로 공부한 후 보는 것을 추천한다.  
Packt 책답게(?) 깊은 내용이 있는 것은 아니고 활용에 중점을 두고 있어서 내 책과 겹치는 부분이 꽤 된다.  