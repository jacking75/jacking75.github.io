---
layout: post
title: C# - SuperSocketLite으로 게임 서버 만들기 - 05 채팅 서버
published: true
categories: [.NET]
tags: c# .NETCore SuperSocket SuperSocketLite socket tcp gameserver network
---
방 구조의 채팅 서버이다.  
  
패킷 처리는 1개의 스레드만으로 처리한다.  
(네트워크 IO는 멀티 스레드)  
  
  
프로젝트 구조는 아래와 같다. 파일 이름을 보면 대충 어떤 일을 맡고 있는지 알 수 있을 것이다  
![supersocket](/images/2019/supersocket/020.png)    
  
  
네트워크 관련 처리를 1개의 스레드에서 하기 위해 MainServer에서 바로 처리하지 않고, 패킷 처리 스레드로 전달한다.  
![supersocket](/images/2019/supersocket/021.png)    
  
  
채팅 서버 핵심 구현은 이 클래스에 다 있다.  
패킷 처리 스레드는 처리할 패킷이 있으면 가져와서 패킷 ID를 보고 이 패킷 ID와 연결된 패킷 처리 함수를 호출한다.  
![supersocket](/images/2019/supersocket/022.png)    
  
  
![supersocket](/images/2019/supersocket/023.png)    
  