---
layout: post
title: 오래된 버전의 .NET Core 삭제 시 .NET Standard 2.0을 인식하지 못하는 문제
published: true
categories: [.NET]
tags: .net c# .netcore
---
컴퓨터 정리를 하다가 여러 버전의 닷넷코어가 설치 되어 있어서 최신 버전(2.2.103)을 제외하고 이전 버전의 닷넷코어를 삭제하였다.  
그런데 Visual Studio에서 '.NET Standard 2.0'을 찾지 못하는 문제가 발생하였다.  
다행히 구글링 해보니 이 문제에 대한 답을 쉽게 찾았다.  
[Can't get .NET Standard 2.0 installed](https://stackoverflow.com/questions/48449553/cant-get-net-standard-2-0-installed )  
  
.NET Standard 2.0은 닷넷코어 최신 버전이 아닌 2.1대 버전을 설치해야 한다고 한다. 그래서 윗 글에서 이야기한 2.1.4 버전을 설치하였다.  
이제 내 컴퓨터에는 2.1.400와 2.2.103 버전이 설치 되어 있다.  
  
Visual Studio를 실행 해보니 .NET Standard 2.0을 인식하였다.    
그런데 이번에는 닷넷코어 2.2 버전을 인식하지 못하고 있다. 2.1만 인식한다.  
  
현재 설치된 모든 닷넷코어 SDK를 다 삭제하고 재부팅.  
2.1.400 버전을 설치하고 재부팅, 2.2.103 버전을 설치하였다.  

.NET Standard 2.0, 닷넷코어 2.2가 잘 인식된다.  
  
  
**닷넷코어 SDK는 설치하면 안 지우는 것이 좋다.**  
