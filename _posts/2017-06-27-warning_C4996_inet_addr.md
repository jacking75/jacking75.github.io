---
layout: post
title: warning C4996 'inet_addr'
published: true
categories: [Network]
tags: Network_Programming socket inet_addr
---
몇 년전에 만들어진 Winsock API를 사용한 코드를 최신 VC++로 빌드하면 아래와 같은 경고를 볼 수 있다.  
<pre>
warning C4996: 'inet_addr': Use inet_pton() or InetPton() instead or define _WINSOCK_DEPRECATED_NO_WARNINGS to disable deprecated API warnings
</pre> 
inet_addr 라는 API는 비 추천이 되었으므로 다른 API를 사용하라는 경고이다.  
inet_addr는 대표적으로 클라이언트에서 소켓으로 서버에 접속하기 위해 서버 주소를 설정할 때 사용한다.  
  
```
#define SERVERIP   "127.0.0.1"
#define SERVERPORT 23452


SOCKADDR_IN serveraddr;
ZeroMemory(&serveraddr, sizeof(serveraddr));
serveraddr.sin_family = AF_INET;

serveraddr.sin_addr.s_addr = inet_addr(SERVERIP); // 경고가 나오는 부분
serveraddr.sin_port = htons(SERVERPORT);
```  
  
추천하는 API를 사용하여 경고를 없애고 싶으면 아래와 같이 바꾼다.  
  
```
SOCKADDR_IN serveraddr;
ZeroMemory(&serveraddr, sizeof(serveraddr));
serveraddr.sin_family = AF_INET;

auto ret = inet_pton(AF_INET, SERVERIP, (void *)&serveraddr.sin_addr.s_addr); 
serveraddr.sin_port = htons(SERVERPORT);
```   
  
