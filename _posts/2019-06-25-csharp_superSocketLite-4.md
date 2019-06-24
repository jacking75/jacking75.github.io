---
layout: post
title: C# - SuperSocketLite으로 게임 서버 만들기 - 04 MultiPortServer
published: true
categories: [.NET]
tags: c# .NETCore SuperSocket SuperSocketLite socket tcp gameserver network
---
서버가 복수의 port 번호를 사용하는 경우에 대한 예제이다.
  
서버는 클라이언트 이외에 서버간 통신을 위해 연결 하기도 한다(보통 분산 서버를 만드는 경우이다).  
  
이 프로젝트는 서버 객체를 2개 생성해서 하나는 클라이언트와 연결, 또 하나는 서버와 연결한다.  
서버와 클라이언트 연결 포트를 다르게 하면 좋은 점은 보안성을 올릴 수 있다.  
서버 간에 주고 받는 패킷은 서버 포트로 연결된 세션만 사용할 수 있다.  
  
![supersocket](/images/2019/supersocket/018.png)  
  
  
참고로 SuperSocket은 아래의 설정 데이터를 보면 알듯이 하나의 서버 객체가 여러 포트를 열어서 사용할 수 있다.    
![supersocket](/images/2019/supersocket/019.png)  
  
  
  
빌드 후 run_MultiPortServer.bat 배치 파일로 실행한다.  