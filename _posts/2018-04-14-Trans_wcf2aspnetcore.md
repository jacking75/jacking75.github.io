---
layout: post
title: WCF와 ASP.NET Core의 성능 비교
published: true
categories: [번역]
tags: 번역 wcf asp.net core
---
[원문](https://www.infoq.com/news/2018/04/wcf2aspnetcore)  
  
Microsoft는 ASP.NET Core 개발에 많은 시간과 개발 능력을 지출했다.  
그 결과, 새로운 기능을 가진 오픈 플랫폼이 탄생하고, 큰 개발자 커뮤니티와의 오픈된 노력에서 혜택을 받아왔다.  
ASP.NET Core는 WCF(Windows Communication Foundation)와 같은 기존 기술보다 성능이 우수한 것으로 기대하고 있다.  
그런데 최근, 그렇지 않다고 생각하고 있는 것 같다. 이에 대해 좀 더 자세히 살펴 보자.  
  
최근 개발자 Erik Heemskerk씨는 ASP.NET Core와 WCF의 성능을 조사한 [기사](https://www.erikheemskerk.nl/benchmark-wcf-webapi-aspnetcore-mvc/)를 썼다.  
그의 실험은 각각의 기술을 사용하여 간단한 프로젝트를 준비하고, "로컬 Web 서버를 시작해 요청 생성, 전송 및 역 직렬화 응답 생성 회신 응답의 직렬화에 걸리는 시간을 측정 한」 것 이라고한다.

놀랍게도 페이로드에 간단한 GUID를 사용한 경우 WCF는 동일한 ASP.NET Core 프로젝트에 비해 약 3배 빨랐다.  
Heemskerk 씨는이 차이는 ASP.NET Core가 JSON을 사용하는데, WCF가 XML로 직렬화하는 것에 있을지도 모른다고 생각하고 ASP.NET Core를 XML로 직렬화 하도록 했다.  
결과는 개선됐지만 여전히 WCF이 훨씬 빨랐다.  
다른 방법을 시도해 보려고 Heemskerk 씨는 페이로드의 크기를 더 현실적인 개체까지 확대하고 ASP.NET Core에서 MessagePack을 사용하도록 했다.  
이렇게 하면 최신 ASP.NET은 WCF 보다 조금 빨라진한다.  
  
그러나 이야기는 이것으로 끝나지 않는다.  
개발자 Josh Bartley씨는 Heemskerk씨의 작업을 [조사하여](https://joshbartley.com/response-is-wcf-faster-than-asp-net-core/), ASP.NET의 결과를 개선하기 위해 다른 변경할 수 있는 것은 없는지 확인했다.  
그의 분석에 따르면, ASP.NET의 벤치 마크는 WCF 코드 작업에 비해 똑같은 작업이 포함 되어 있는 것은 아니었다.  
  
따라서 ASP.NET Core는 성능면에서 늦는 것은 아니다.  
만약 첫 성능이 기대를 충족 해야 한다면 최고의 성능을 내기 위해 어떤 분석이 필요할 것이다.  
성능 개선을 시도 할 때 큰 교훈은 바로 이곳을 벤치마크로 선택하고, 적절한 위치의 코드가 개선 되도록 하는 것이다.    
  
