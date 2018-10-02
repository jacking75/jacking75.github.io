---
layout: post
title: .NET Core - RID 카탈로그
published: true
categories: [.NET]
tags: .net c# .netcore RID
---
RID는 Runtime IDentifier(런타임 식별자)의 약어이다.  
RID 값은 응용 프로그램을 실행하는 대상 플랫폼을 식별하는데 사용된다.  
[Docs의 설명](https://docs.microsoft.com/ko-kr/dotnet/core/rid-catalog)  

예)
```
dotnet publish --runtime centos.7-x64
```
\bin\Debug\netcoreapp2.1\centos.7-x64\publish  
에 파일이 만들어진다.

