---
layout: post
title: SignalR .NET Core - Realtime cross-platform open web communication
published: true
categories: [.NET]
tags: c# .net SignalR
---
MS Build2017에서 발표된  
[SignalR .NET Core: Realtime cross-platform open web communication](https://channel9.msdn.com/Events/Build/2017/B8078)  
의 간단 요약  

- ASP.NET Core에 맞추어서 재 작성하고, 재 설계  
- HTTP 이외에서도 이용(AMQP,MQTT,TCP)  
- 탈 jQuery, WebWorker 에서의 이용도 생각
- JSON & ProtocolBuff protocol  
- Redis, Service Bus, SQL Server(TBD)에 의한 스케일아웃  
- .NET Standard, TypeScript, C++, Java, Swift 용 클라이언트 라이브러리  
- 바이너리 지원  
- .NET Core 2.0에 맞추어서 Preview, 가을쯤에 GA