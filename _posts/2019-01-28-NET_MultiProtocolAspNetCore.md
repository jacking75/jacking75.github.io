---
layout: post
title: ASP.NET Core 하나의 호스트에서 http와 socket 통신 같이 하기
published: true
categories: [.NET]
tags: .net c# .netcore MultiProtocolAspNetCore tcp socket Kestrel
---
[Github](https://github.com/davidfowl/MultiProtocolAspNetCore )  

ASP.NET Core의 Kestrel을 사용하여 TCP Socket 프로그래밍을 할 수 있다.  
MultiProtocolAspNetCore 프로젝트의 Program.cs 파일의 내용은 아래와 같다.  
```
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
	WebHost.CreateDefaultBuilder(args)
		.UseKestrel(options =>
		{
			// TCP 8007
			options.ListenLocalhost(8007, builder =>
			{
				builder.UseConnectionHandler<MyEchoConnectionHandler>();
			});

			// HTTP 5000
			options.ListenLocalhost(5000);

			// HTTPS 5001
			options.ListenLocalhost(5001, builder =>
			{
				builder.UseHttps();
			});
		})
		.UseStartup<Startup>();
```
  
ASP.NET Core이니 http는 당연하고,  http 아래 단의 socket 통신도 직접 컨트롤 할 수 있다.  
위 파일에서 http 부분을 제거하고 좀 더 기능을 구현하면 고성능 비동기 IO tcp socket 서버를 쉽게 만들 수 있다.   
  
위 프로젝트는 TCP Socket으로 Echo 기능을 구현하고 있다.  

