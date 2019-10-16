---
layout: post
title: IOCP 관련 확장 함수 사용 준비
published: true
categories: [Network]
tags: iocp windows network c++ 네트워크
--- 
오랜된 Windows(아마 Windows Vista 이전)에서는 Winsock 확장 함수를 사용하기 위해서는 확장 함수의 함수 포인터를 가져와야 사용할 수 있었다.  
 
 
예를들면 AcceptEx의 경우는 아래처럼 해야 사용할 수 있었다.  
```
static LPFN_ACCEPTEX mFnAcceptEx;

GUID guidAcceptEx = WSAID_ACCEPTEX ;
if (SOCKET_ERROR == WSAIoctl(mListenSocket, SIO_GET_EXTENSION_FUNCTION_POINTER,
    &guidAcceptEx, sizeof(GUID), &mFnAcceptEx, sizeof(LPFN_ACCEPTEX), &bytes, NULL, NULL)) {
    return false;
}

mFnAcceptEx(sListenSocket, 
                sAcceptSocket, 
                lpOutputBuffer, 
                dwReceiveDataLength,
                dwLocalAddressLength, 
                dwRemoteAddressLength, 
                lpdwBytesReceived, 
                lpOverlapped);
```
  
  
그러나 요즘 Windows에서는 아래처럼 확장 함수를 바로 사용할 수 있다.    
먼저 아래의 lib 파일을 포함해야 한다.  
```
#pragma comment(lib, "mswsock.lib")  
```
  
AcceptEx 함수를 바로 사용한다.  
```  
AcceptEx(sListenSocket, 
            sAcceptSocket, 
            lpOutputBuffer, 
            dwReceiveDataLength,
            dwLocalAddressLength, 
            dwRemoteAddressLength, 
            lpdwBytesReceived, 
            lpOverlapped);
```  


