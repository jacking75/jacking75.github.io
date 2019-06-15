---
layout: post
title: C# - SuperSocketLite으로 게임 서버 만들기 - 01
published: true
categories: [.NET]
tags: c# .NETCore SuperSocket SuperSocketLite socket tcp gameserver network
---
C#에서 가장 유명한 네트워크 라이브러리로 SuperSocket 이라는 것이 있다.  
앞으로 여러 번을 나누어서 SuperSocket을 사용하여 게임 서버를 만드는 방법을 설명하려고 한다.  
  
## SuperSocket 이란?  
SuperSocket의 코드는 Github에 있다.  
https://github.com/kerryjiang/SuperSocket  
현재는 2.0 버전을 작업 중으로 올해말 정도에 첫 번째 버전이 나올 예정이라고 한다.  
그래서 지금 안전하게 사용하기 위해서는 1.6 버전을 사용해야 한다.    
[1.6 버전 브랜치 ](ttps://github.com/kerryjiang/SuperSocket/tree/v1.6 )  
1.6 버전은 닷넷프레임워크와 Mono만 지원한다. 닷넷코어는 지원하지 않는다.  
  
1.6에서 닷넷코어를 지원하지 않고, 2.0 버전은 아직 개발 중이라서 나는 1.6 버전을 수정해서 닷넷코어에서 동작하도록 수정을 하였다.     
[SuperSocketLite](https://github.com/jacking75/SuperSocketLite )  
  
  
## SuperSocketLite
기존의 SuperSocket을 닷넷코어에서 동작하도록 변경한 것이다.  
처음 계획으로는 기존의 SuperSocket에서 게임 서버 개발에 사용하지 않는 기능들을 삭제해서 구조를 간단하게 만들 목적이라서 `Lite`라는 이름을 붙였다.  
그러나 뒤에 기존에 SuperSocket을 사용한 서버 프로그램을 쉽게 포팅하기 위해서 닷넷코어에서 지원하지 않는 기능들만 삭제하였다.  
  
기존의 SuperSocket을 사용할 때 AppDomain과 Dlr, Managent 기능을 사용하지 않았다면 거의 코드 변경 없이 SuperSocketLite를 사용할 수 있다.  
  
  
## 문서
앞서 이야기 했듯이 SuperSocket을 거의 그대로 가져온 것이라서 문서도 기존 SuperSocket 문서를 보면 된다.  
SuperSocketLite 저장소의 /Docs/SuperSocket-1_6-Doc/  디렉토리에 원본 문서를 pdf 버전으로 보관해 두었다.  
  
2016년에 본인이 KGC 라는 게임 개발 컨퍼런스에서 SuperSocket을 주제로 강연을 한 적이 있다.  
[KGC 2016 'SuperSocket' 관련 강연](https://github.com/jacking75/conf_kgc2016_SuperSocket )  
  
위 문서들을 참고하면 구조나 사용방법을 배울 수 있다.   
  
  
## Samples
/Samples 디렉토리에 간단한 서버와 채팅 서버 코드가 있다.    
- BinaryPacketServer
    - 바이너리 프로토콜을 사용하여 서버/클라이언트가 통신하는 서버
- SimpleUDPServer
    - UDP를 사용하는 서버
- SwitchReceiveFilter
    - SuperSocket으 내장 프로토콜을 실행 중 변경하는 서버
- sendFailTestServer
    - 클라이언트가 Receive를 하지 않을 때 서버에서 Send 오류를 잘 처리하는지 테스트 하는 서버
- Chat
    - 방 구조의 채팅 서버
  
<br/>  
  
문서와 예제 코드가 있어서 어렵지 않게 사용 방법을 배울 수 있다고 생각하지만 어려운 사람들도 있으리라 생각한다.  
그래서 앞으로 몇 번에 걸쳐서 Tutorials 디렉토리에 있는 코드를 중심으로 사용 방법에 대해서 설명하겠다.  
  