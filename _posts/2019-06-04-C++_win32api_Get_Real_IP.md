---
layout: post
title: C++ - 컴퓨터의 실제 IP 얻기  (code)
published: true
categories: [C++]
tags: c++ code GetLocalIP
---
  
```
#include <ws2tcpip.h>

bool ServerInfoMgr::GetLocalIP( string& strIP )
{
    char host_name[256];
    if (gethostname(host_name, 256) == SOCKET_ERROR)
         return false;

   struct addrinfo hints, *res = NULL;
   char *szRemoteAddress=NULL, *szRemotePort=NULL;
   int rc; memset( &hints, 0, sizeof(hints) );
   hints.ai_family = AF_INET;
   hints.ai_socktype = SOCK_STREAM;
   hints.ai_protocol = IPPROTO_TCP;

   rc = getaddrinfo( host_name, szRemotePort, &hints, &res );
   if( rc == WSANO_DATA ) { return false; }

   char szIP[MAX_IP_STRING_LENGTH] = {0, };
   sprintf_s( szIP, MAX_IP_STRING_LENGTH, "%d.%d.%d.%d",
               (unsigned char)res->ai_addr->sa_data[2],
              (unsigned char)res->ai_addr->sa_data[3],
              (unsigned char)res->ai_addr->sa_data[4],
              (unsigned char)res->ai_addr->sa_data[5] );
             
   strIP = szIP;
   return true;
} 
```
  