---
layout: post
title: .NET Core - Improving .NET Core Kestrel performance using a Linux-specific transport
published: true
categories: [.NET]
tags: .net c# .netcore Kestrel
---
[이 글을 구글 번역을 통해 일부 정리했다](https://developers.redhat.com/blog/2018/07/24/improv-net-core-kestrel-performance-linux/)  

성능은 핵심 기능입으로 많이 최적화되어 지속적으로 벤치마킹 하고 있다.   
Kestrel은 HTTP 서버의 이름이다.  
이 블로그 포스트에서 우리는 Kestrel의 네트워킹 레이어를 리눅스 관련 구현으로 대체하고, 기본 구현 방식과 비교하여 벤치마킹 할 것이다.  
TechEmpower 웹 프레임 워크 벤치 마크는 네트워크 계층 성능을 비교하는 데 사용된다.  
  

## 전송 추상화
Kestrel은 전송 추상화 덕분에 네트워크 구현을 대체 할 수 있다.  
ASP.NET Core 1.x는 libuv을 네트워크 구현에 사용한다.  
libuv는 Node.js의 토대가 되는 비동기 I/O 라이브러리이다. l 
ASP.NET Core 2.0은 Kestrel에서 전송 추상화를 도입하여 libuv을 소켓 기반 구현으로 변경할 수 있게 되었다.   
버전 2.1에서는 소켓 구현에 더 많은 최적화가 이루어졌으며 소켓 전송은 Kestrel에서 기본 값이 되었다.  

전송 추상화를 사용하면 다른 네트워크 구현을 연결할 수 있다. 예를 들어 Windows RIO 소켓 API 또는 사용자 공간 네트워크 스택을 활용할 수 있다.  
이 블로그 게시물에서는 Linux 관련 전송을 살펴 보겠다. 이 구현은 libuv/ Sockets 전송을 직접 대체하는데 사용할 수 있다.  
특권 기능이 필요 없으며, 예를 들어 Red Hat OpenShift 에서 실행되는 경우와 같이 제한된 컨테이너에서 작동한다.  

향후 버전의 경우, Kestrel은 비 HTTP 서버의 기반으로 더 유용 해 지려고 한다. 전송 및 관련 추상화는 해당 프로젝트의 일부로 변경된다.  


## 벤치 마크 소개
Microsoft는 ASP.NET Core 스택을 지속적으로 벤치 마킹 하고 있다. 결과는 https://aka.ms/aspnet/benchmarks 에서 볼 수 있다.  
벤치 마크에는 TechEmpower 웹 프레임 워크 벤치 마크의 시나리오가 포함된다.  

TechEmpower 벤치 마크에 대해 간략하게 설명하겠다.  

Fortune 테스트 유형은 ORM (object-relational mapper)과 데이터베이스를 사용하기 때문에 가장 흥미롭다. 이것은 웹 응용 프로그램/서비스의 일반적인 유스 케이스이다. 이 시나리오에서는 이전 버전의 ASP.NET Core가 제대로 작동하지 않았다. ASP.NET Core 2.1은 스택 최적화와 PostgreSQL 드라이버 덕분에 성능이 크게 향상되었다.  

다른 시나리오는 일반적인 응용 프로그램을 대표하지 않는다. 이들은 스택의 특정 측면을 강조한다. 이들 유스 케이스와 밀접하게 일치하는지 살펴 보는 것은 흥미로울 수 있다. 프레임 워크 개발자의 경우 스택 최적화를 위한 기회를 파악하는데 도움이 된다.  

예를 들어, 일반 텍스트 시나리오를 고려하자. 이 시나리오는 I/O 작업이나 계산을 수행 할 필요없이 서버가 응답을 알고 있는 16개의 요청을 연속적으로( 파이프 라인 방식으로 ) 전송하는 클라이언트를 포함한다. 이것은 일반적인 요청을 나타내는 것은 아니지만 HTTP 요청을 구문 분석하기 위한 좋은 스트레스 테스트이다.  

각 구현에는 클래스가 있다. 예를 들어 ASP.NET Core Plaintext에는 플랫폼 , 마이크로 및 전체 구현이 있다. 전체 구현은 MVC 미들웨어를 사용한다. 마이크로 구현은 파이프 라인 레벨에서 구현되며 플랫폼 구현은 직접 Kestrel 위에 구축된다. 플랫폼 클래스는 엔진이 얼마나 강력한지에 대한 아이디어를 제공하지만 응용 프로그램 개발자가 프로그램을 작성하는데 사용되는 API는 아니다.  

벤치 마크 결과에는 대기 시간 탭이 포함된다. 일부 구현은 초당 매우 많은 수의 요청을 처리하지만 상당한 대기 시간 비용이 소요된다.  


## Linux 전송
다른 구현과 마찬가지로 Linux 전송은 논블럭킹 소켓과 epoll을 사용한다. .NET 코어의 Socket은 eventloop 관리(C#) 코드에서 구현된다. 이것은 네이티브 라이브러링ㄴ libuv의 libuv 루프와 다르다.  

두 개의 Linux 관련 기능이 사용된다. SO_REUSEPOR로 T커널이 여러 스레드에서 허용된 연결을 로드 밸런싱하고, Linux AIO API를 사용하여 호출을 일괄적으로 보내고 받을 수 있다.  


## Benchmark
벤치마크 결과는 원문을 보자.  


## Linux 전송 사용
Kestrel Linux 전송은 실험적인 구현이다. myget.org에 2.1.0-preview1 로 게시된 패키지를 사용하여 시험해 볼 수 있다. 이 패키지를 사용 하면이 GitHub 문제를 사용하여 의견을 제공하고 (보안) 문제를 알릴 수 있다. 사용자의 의견을 토대로 nuget.org에 게시된 지원되는 2.1 버전을 유지하는 것이 합당한지 알 수 있다.  
[GitHub])(https://github.com/redhat-developer/kestrel-linux-transport)  

myget 피드를 NuGet.Config 파일에 추가하려면 다음을 수행한다.  
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
	<add key="rh" value="https://www.myget.org/F/redhat-dotnet/api/v3/index.json" />
  </packageSources>
</configuration>
```  

그리고 csproj 파일에 패키지 참조를 추가한다.  
```
<PackageReference Include="RedHat.AspNetCore.Server.Kestrel.Transport.Linux" Version="2.1.0-preview1" />
```  

그런 다음 WebHost를 생성할 UseLinuxTransport를 호출한다. WebHostProgram.cs:  
```
public static IWebHost BuildWebHost(string[] args) =>
        	WebHost.CreateDefaultBuilder(args)
            	.UseLinuxTransport()
            	.UseStartup()
            	.Build();
```  

UseLinuxTransport은 비 Linux 플랫폼에서 호출해도 안전하다 . 이 메소드는 응용 프로그램이 Linux 시스템에서 실행될 때만 전송을 변경한다.  


## 결론
이 블로그 포스트에서 Kestrel과 전송 추상화 네트워크 구현을 대체하는 방법을 배웠다. TechEmpower 벤치 마크를 자세히 살펴본 후 CPU 및 네트워크 제한이 벤치 마크 결과에 미치는 영향에 대해 살펴 보았다. 리눅스 특유의 트랜스포트는 디폴트의 out-of-the-box 구현에 비해 상당한 이득을 줄 수 있음을 알았다.  
