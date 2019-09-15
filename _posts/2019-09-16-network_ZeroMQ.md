---
layout: post
title: ZeroMQ
published: true
categories: [Network]
tags: zeromq mqtt
---
## 소개
-  공식 사이트  http://www.zeromq.org/
-  Github https://github.com/zeromq
- 예제 https://github.com/imatix/zguide
-  N-N 통신을 구현하는 socket API풍의 경량 메시지 라이브러리
-  자동적으로 재접속이나 메시지 큐잉을 해 준다
-  복수의 메시징 패턴 이라는 것을 조합하여 유연한 메시징 배신을 할 수 있다.
-  오픈소스
-  Windows 및 리눅스 계열 지원
-  C++로 만들었음
-  리눅스의 네트웍 라이브러리의 표준이 되자는 목표를 가지고 시작
-  C#, Java, 파이썬 등의 다양한 언어 지원
  
  
## 특징
-  퍼포먼스
    - ZeroMQ는 정말 빠르다. 
    - 그것은 대부분의 AMQP(역자: Advanced Message Queuing Protocol )들 보다 단위가 다를 정도로 빠르다. 이러한 퍼포먼스는 다음과 같은 테크닉들 때문에 가능하다:
    - AMQP처럼 과도하게 복잡한 프로토콜이 없다.
    - 신뢰성 있는 멀티캐스트나 Y-suite IPC 전송 같은 효율적인 전송을 활용한다.
    - 지능적인 메시지 묶음을 활용한다. 이것은 0MQ로 하여금 프로토콜 오버헤드뿐만 아니라 시스템 호출을 줄여서 TCP/IP를 효율적으로 활용하게 한다.
-  단순성
    - API는 믿을 수 없을 정도로 간단하다. 그렇기에 소켓 버퍼에 계속 '값을 채워' 주어야 하는 생 소켓 방식에 비교하면 메시지를 보내는 것이 정말로 단순하다. ZeroMQ에서는 그냥 비동기 send 호출을 부르기만 하면, 메시지를 별도의 스레드의 큐에 넣고 필요한 모든 일을 해준다. 이러한 비동기 특성이 있기에 당신의 애플리케이션은 메시지가 처리되기를 기다리며 시간 낭비하지 않아도 된다. 0MQ의 비동기 특성은 이벤트 중심의 프레임웍에도 최적이다.
    - ZeroMQ의 단순한 와이어 프로토콜은 다양한 전송 프로토콜이 사용되는 요즘에 적합하다. AMQP를 쓴다면 그 위에 또 다른 프로토콜 레이어를 쓴다는 것은 좀 이상하게 느껴진다. 0MQ는 메시지를 그냥 Blob으로 보기에 당신이 메시지를 어떻게 인코드하든 상관없다. 단순히 JSON 메시지들을 보내던지, 아니면 BSON, Protocol Buffers나 Thrift 같은 바이너리 방식 메시지들도 괜찮다.
-  확장성
    - ZeroMQ 소켓들은 저수준처럼 보이지만 사실은 다양한 기능들을 제공한다. 예를 들어 하나의 ZeroMQ 소켓은 복수의 접점을 가질 수 있으며 그들 간에 자동으로 메시지 부하 분산을 수행한다. 또는 하나의 소켓으로 복수의 소스에서 메시지들을 받아들이는 게이트 역할을 할 수도 있다.
    - ZeroMQ는 '브로커없는 설계' 방식을 따르기에 전체의 실패를 초래하는 단일 위치가 존재하지 않는다. 이것과 앞에서 설명한 단순함 그리고 퍼포먼스를 조합하면 당신의 애플리케이션에 분산처리 기능을 넣을 수 있다.
  
  
## 라이브러리 기능
-  socket API와 비슷한 C API를 가진다. socket은 zeromq의 socket을 가리킨다.
-  zeromq는 컨텍스트라는 것을 통해서 통신을 한다. 1 컨텍스트에 I/O 스레드가 하나 할당 된다. 기본 하나의 프로세스에 하나의 컨텍스트로 OK. 복수의 context를 가질 수 있으면 이 경우 같은 개수의 I/O 스레드가 실행된다.
-  zeromq의 socket은 프로세스 내 통신(스레득간 통신 등), 프로세스간 통신, TCP, UDP 멀티캐스트를 랩핑한 것이 있다. icp:my.server나, tcp:127.0.0.1:1234와 같이 통신할 곳을 설정할 수 있다.
-  통신에서는 메시지를 주고 받는다. 메시지는 복수는 프레임에서 된다. 하나의 프레임은 하나의 길이가 있는 바이너리를 가진다.
-  하나의 socket에서 N-N 통신을 할 수 있다.
-  하나의 노드는 복수의 socket을 가질 수 있다.
-  zeromq의 socket은 진짜 socket과는 다르다. 클라이언트를 먼저 실행 할 수도 있다. 
-  접속에는 메시징 패턴이라는 것을 설정 되고 있다. 패턴에 의해서 메시지 송수신 방법이 달라진다.
-  서버는 데이터 송수신, 클라이언트 데이터 수신자자 아니다. 어디까지나 메시징 패턴에 기초를 둔 메시지 통신을 한다.
-  노드는 동적으로 추가 할 수 있다.
-  메시징 패턴이 준비한 메타 정보를 제외하고 zeromq는 메시지 내용을 관여하지 않는다.
  
  
## ZeroMQ로 메시징 레이어 구현하기
-  전송 방식 정하기
    - INPROC  프로세스내 전송 모델
    - IPC  프로세스 간 전송 모델
    - MULTICAST  PGM를 통한 멀티캐스트 (UDP로 싸여지기도 함)
    - TCP  네트워크 기반의 전송
-  하부 구조 잡기
    - QUEUE  요청/답변(Request/Reply) 메시징 패턴을 위한 발송자
    - FORWARDER  발행/구독(Publish/Subscribe) 메시징 패턴을 위한 발송자
    - STREAMER  파이프라인의 단방향 메시징 패턴을 위한 발송자
-  메시지 패턴을 선택하기
    - REQUEST/REPLY  양방향, 부하 분산과 상태 기반
    - PUBLISH/SUBSCRIBE  다수의 구독자들에게 한번에 발행
    - UPSTREAM / DOWNSTREAM  파이프라인의 접점들에 단일 방향 데이터 배포
    - PAIR  두 피어들 간 배타적 커뮤니케이션
  
  
## ZeroMQ via C#
- 닷넷에서 ZeroMQ 사용하기
    - ZeroMQ via C#: Introduction 
    - http://www.codeproject.com/Articles/488207/ZeroMQ-via-Csharp-Introduction
- Github https://github.com/zeromq/clrzmq
-  Sample https://github.com/imatix/zguide/tree/master/examples/C%23
-  설치는 NuGet으로 가능하다.
    - 닷넷에서 사용할 때 libzmq.dll(C++ 소스로 컴파일한 것)과 clrzmq.dll(libzmq.dll을 닷넷으로 랩핑한 것) 둘 다 필요하다. 당근 NuGet을 통하면 다 셋팅된다
-  Mono
    - NuGet 설치를 위해서는 2.10.7 버전 이상, 2.6 버전 이상에서는 수동으로 설치해야 한다.
  
  
## 참고 사이트
-  ZeroMQ에 대한 소개글 번역  http://blog.naver.com/haje01/130133918260
-  About zeroMQ  http://www.open-ware.co.kr/html/about_zeromq.html
-  ØMQ - The Guide  http://kr.zeromq.org/guide:start
