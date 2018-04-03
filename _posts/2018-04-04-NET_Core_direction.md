---
layout: post
title: .NET Core 2.1은 어디로 향하고 있는가?
published: true
categories: [.NET]
tags: .net c# .netcore ubuntu
---
[원문](https://www.infoq.com/news/2018/02/netcore2.1-direction)   
   
Microsoft의 Scott Hunter씨는 .NET Core 2.1에서의 동사의 계획을 발표했다.  
CLI 툴이 개발자의 편리한 사용을 대폭 향상시키는 것으로 보인다.  
Microsoft는 매일 50만명에 가까운 개발자의 적극적인 이용을 감시할 수 있다고 Hunter씨는 말했다.  
Microsoft 통계에 따르면 2017년 9월 .NET Core 2의 이용률은 .NET Core 1.x를 넘어섰다.  
Microsoft는 .NET Core 2.1에서 주력하는 여러가지 주제 가운데 빌드 성능의 고속화, 내부 엔지니어링 시스템의 고속화나 .NET Framework 와의 호환성 향상이 있다.  
모든 .NET Core 프로젝트는 2.1에서 빌드 시간이 고속화할 것이며, Microsoft의 사전 벤치 마크에서는 큰 프로젝트 일수록 큰 이점을 얻을 수 있었다.  
  
.NET Core 2.1은 "minor-version roll-forward"로 불리는 체제 호환에 대한 접근을 처음 공개할 예정이다.  
이 기능으로 애플리케이션의 런타임 대응에 걸리는 노력을 줄일 수 있다.  
이것은 .NET Core 2.1을 이용하는 애플리케이션은 수정 없이 장래의 마이너 버전(2.2, 2.3, etc.)에서 실행할 수 있음을 의미한다.  
  
Ready To Run(R2R) 프리 컴파일된 어셈블리에서는 설치 크기가 줄어들 예정이다.  
.NET Core 2가 채용한 접근법은 기동 시간 단축은 좋았지만 일단 앱이 실행 개시하면 이것에 의해서 얻는 이익은 적었다.  
  
.NET Core를 지원하는 명령 라인 툴도 개선 예정이다.  
개발자의 편리성을 높일 목적으로 개발 툴의 패키지화와 설치를 지원하기 위해 .NET Core 2.1에 몇 가지 새로운 명령이 추가된다.  
dotnet pack은 배포 시에 필요한 어셈블리를 패키지하고, dotnet install tool exampleApp은 실행 시 사용자 디렉토리의 .dotnet\tools 폴더에 설치한다.  
이곳은 자동적으로 패스에 추가되기 때문에 로컬 디렉터리에 관계 없이 새로운 실행 파일을 사용할 수 있다.  
  
이들 어플리케이션 전용의 추가 개선으로 dotnet publish 를 사용하는 것으로 패키지화 할 수 있다.  
.NET Core 2.1에서는 작성되는 패키지에 대해 기본적으로 런타임의 최신 패치가 포함된다.  
  
공식 스케줄 새로 나오지 않지만 Hunter씨는 그의 팀이 NET Core 2.1 프리뷰 릴리스에 참여했으며, 2월에 공개할 예정이다.  
2차 프리뷰는 3월이 되고, Release Candidate는 4월을 예정하고 있다.  
2018년 상반기쯤에서 공식적으로 나올 예정이다.  
    
