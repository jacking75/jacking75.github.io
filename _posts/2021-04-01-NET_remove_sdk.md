---
layout: post
title: C# - 복수 설치된 .NET Core SDK 삭제
published: true
categories: [.NET]
tags: .net c# .netcore sdk delete
---  
dotnet-core-uninstall 툴을 설치한다  
https://docs.microsoft.com/ko-kr/dotnet/core/additional-tools/uninstall-tool?tabs=windows  
  
  
아래 명령어로 삭제할 수 있는 SDK 버전을 본다   
```
dotnet-core-uninstall list
```    
  
  
아래 명령어로 삭제할 수 있는 버전을 본다  
```
dotnet-core-uninstall dry-run --all --sdk 
```  
  
  
아래 명령어로 삭제한다  
```
dotnet-core-uninstall remove --all --sdk
```  
    
	
**삭제 시에는 관리자 권한이 필요하다**  

  
   