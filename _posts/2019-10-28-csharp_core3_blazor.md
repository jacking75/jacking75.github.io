---
layout: post
title: C# - .NET Core 3.0에서 Blazor가 정식 버전으로
published: true
categories: [.NET]
tags: .net c# .netcore Blazor
---
[출처](https://www.publickey1.jp/blog/19/blazorcnet_core_30web.html )  
  
마이크로 소프트의 Web 응용 프로그램 프레임워크 "Blazor"가 .NET Core 3.0의 출시와 동시에 정식으로 출시 되었다.  
  
Blazor는 JavaScript와 React나 Angular 등의 프레임워크 대신 C# 및 .NET Core 프레임워크 등을 이용하여 Web 애플리케이션의 개발을 가능하게 하는 프레임워크이다.  
  
Blazor는 WebAssembly에 .NET 프레임워크 및 런타임을 구현함으로써, Web 브라우저에서 .NET용 Web 응용 프로그램을 구현하는 프레임워크로 등장했다.   
  
이 구현은 "Blazor WebAssembly" 또는 "Client-side Blazor"로 현재도 개발이 진행 되고 있으며 내년 2020년 5월에 등장 할 전망이다.  
  
이번에 정식 버전이 된 것은 "Server-side Blazor" 또는 "Blazor Server」 나 「ASP.NET Core 3.0 Blazor」라는 구현으로  ASP.NET Core 3.0과 Blazor을 이용한 서버 측에서 실행된다  
  
Server-side Blazor에서는 Web 브라우저 전용 JavaScript 구성 요소를 로드한다. 이 구성 요소는 "ASP.NET SignalR"이라는 기술로 서버와 클라이언트 간의 통신을 실시하면서, Web 브라우저 상에서 DOM 조작 등을 실시함으로써 화면 표시를 한다.  
  
Client-side Blazor와 Server-side Blazor는 응용 프로그램이 실행되는 위치가 다를뿐으로 응용 프로그램의 코드는 공통적으로 될 예정이다.  
  
Client-side Blazor는 오프라인에도 대응, Server-side Blazor는 로딩 시간이 짧은 등 각각의 특성은 다르다.  
  
Sever-side Blazor의 정식 출시는 9월 24일에 열린 .NET Core 3.0의 정식 출시와 동시에 행해졌다.  

Blazor는 Web 애플리케이션의 프런트엔드 주위를 포함하여 .NET 기술로 구현 가능하게 하는 새로운 대안을 제공 할 수 있다. 내년 5월에 등장 예정인 Blazor WebAssembly은 이것을 더 대담한 방법으로 실현하는 것으로서 기대된다.  
  
[ASP.NET Core Blazor 소개](https://docs.microsoft.com/ko-kr/aspnet/core/blazor?WT.mc_id=DT-MVP-4024485 )  