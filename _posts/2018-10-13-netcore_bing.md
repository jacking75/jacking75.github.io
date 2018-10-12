---
layout: post
title: .NET Core - .NET Core 2.1에 의한 Microsoft Bing의 고속화
published: true
categories: [.NET]
tags: .net c# .netcore RID
---
[출처](https://www.infoq.com/news/2018/08/bing-speedup-dotnet-core-2.1)   

Microsoft의 검색 엔진 Bing은 .NET Core 2.1로 마이그레이션 후 [내부 서버의 대기 시간이 34% 감소했다](https://blogs.msdn.microsoft.com/dotnet/2018/08/20/bing-com-runs-on-net-core-2-1/). Microsoft의 엔지니어인 Mukul Sabharwal 씨에 의하면, 대다수는 .NET 커뮤니티의 기여 덕분이다.  

그에 따르면 .NET Core에 대한 많은 개선이 속도에 영향을 주었다. [문자열 일치의 벡터화](https://github.com/dotnet/coreclr/pull/16994), 새로운 Span<T>형식을 사용한 string.IndexOf/LastIndexOf 에 의해 HTML 렌더링과 조작의 고속화, EqualityComparer를 위한 탈 가상화 EqualityComparer.Default에 의한 딕셔너리 고속화, 병렬 GC에 대한 기록 시계의 CPU 사용 감소 등이다.  

대부분의 개선은 .NET 커뮤니티에서 이루어졌다.  
그러나 GitHub의 풀 요청의 대부분은 Microsoft 직원에 의해 만들어지고 있다.   
Bing을 .NET Core 2.1로 마이그레이션 할 수 있는데는 두 가지 요인이 있었다.  
하나는 ReadyToRun 이미지의 지원이다. 이것에 의해 배포 전에 JIT 컴파일을 할 수 있다.  
이것이 없으면, 각 실행 시스템에서 JIT 컴파일을 할 필요가 있다. 그렇게되면, Bing을 구성하는 서버의 대수가 많기 때문에 중요한 서비스 용량 저하가 일어나 버린다.  
NET Core의 crossgen 도구를 사용하여 미리 컴파일하여 이미지를 배포 할 수있게 되었다.  
두 번째 요소는 .NET Standard 2.0 이다. 이것은 32000개 이상의 API 컬렉션이며, 이것을 사용하여 개발자는 여러 플랫폼에서 .NET Core 2.1으로의 전환을 쉽게 구현할 수 있다.  

그리고 마지막으로 그가 강조한 것은 지속적인 통합 파이프 라인에서 Bing 응용 프로그램 중 .NET Core 런타임을 xcopy를 사용하여 배포 하는 것의 중요성이다. 이것에 의해 .Net Core 2.1이 공식 출시된지 불과 이틀 만에 Bing을 2.1으로 마이그레이션 할 수 있었다.  

InfoQ이 이전에 보도 한 바와 같이, .NET Core 2.1의 최대의 세일즈 포인트 중 하나는 속도 개선이다 . 또한 [자체 내장 배치](https://docs.microsoft.com/en-us/dotnet/core/deploying/runtime-patch-selection) 라는 형태로 새로운 배치 옵션을 제공한다.  
성능면에서는 .NET Core 2.1은 새로운 System.Span<T> 형식을 지원하고 있다. 이것은 F# 4.5에서 추가된 것이다.  
또한 JIT 컴파일러에는 많은 최적화가 이루어지고 있다. 만약이 속도 개선에 대해 깊이 알고 싶다면 [Microsoft 엔지니어 Stephen Toub 씨의 글을 읽는 것](https://blogs.msdn.microsoft.com/dotnet/2018/04/18/performance-improvements-in-net-core-2-1/)을 추천한다 .  


