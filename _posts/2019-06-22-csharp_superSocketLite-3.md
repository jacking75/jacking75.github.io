---
layout: post
title: C# - SuperSocketLite으로 게임 서버 만들기 - 02) 에코 서버 2
published: true
categories: [.NET]
tags: c# .NETCore SuperSocket SuperSocketLite socket tcp gameserver network
---
EchoServer를 좀 더 고도화 한 것이다.  
  
SuperSocketLite 프로젝트를 추가해서 참조하지 않고, SuperSocketLite를 빌드한 dll 파일을 참조한다.  
00_superSocketLite_libs 디렉토리에 있는 dll을 추가한다.  
![supersocket](/images/2019/supersocket/010.png)  
  
또 Nuget으로 `System.Configuration.ConfigurationManager`을 추가한다  
![supersocket](/images/2019/supersocket/011.png)  
  
  
서버 옵션을 프로그램 실행 시 인자로 받는다.  
```
dotnet netcoreapp2.2\EchoServerEx.dll --port 32452 --maxConnectionNumber 32 --name EchoServe
```  
  
인자 파싱은 CommandLineParser 라는 라이브러리를 사용하였고, Nuget으로 받는다.  
![supersocket](/images/2019/supersocket/012.png)    
  
  
NLog 라는 로그 라이브러리를 사용한다. (로그를 콘솔과 파일로 남긴다)  
![supersocket](/images/2019/supersocket/014.png)    
  
NLog.config에 로그 설정 정보가 있다.  
![supersocket](/images/2019/supersocket/015.png)    
  
이 파일은 실행 파일과 같은 위치에 있어야 한다.  
![supersocket](/images/2019/supersocket/016.png)    
  
로그 라이브러리로 NLog 사용은 아래처럼 한다  
![supersocket](/images/2019/supersocket/017.png)    
  
  
나머지는 앞의 EchoServer와 같다.  
빌드 후 run_EchoServerEx.bat 배치 파일로 실행한다.  