---
layout: post
title: C# - Ubuntu에서 snap으로 dotnet-sdk 설치하기
published: true
categories: [.NET]
tags: .net c# netcore ubuntu snap
---
아래 명령어를 실행한다.  
```
sudo apt search dotnet-sdk
```
  
dotnet-sdk 명령어  
```  
dotnet-sdk.dotnet
```
   
   
만약 「dotnet-sdk」 명령어로 「dotnet-sdk.dotnet」을 실행하고 싶다면 아래 명령어를 실행한다.    
```
sudo snap alias dotnet-sdk.dotnet dotnet
```
  