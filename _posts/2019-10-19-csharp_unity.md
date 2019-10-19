---
layout: post
title: C# - Unity와 .NET
published: true
categories: [.NET]
tags: .net c# .netcore unity
---
[출처](https://speakerdeck.com/ryotamurohoshi/unitykai-fa-zhe-nichuan-etai-dot-netfalsekoto )  
  
2016년 여름  
- Unity가 .NET Foundation에 참가
  
2017년 여름 Unity 2017.1 출시 
- .NET 3.5 Equivalent에 더하여 .NET 4.6 Equivalent가 Experimental로 선택할 수 있게 됨
  
2018년 초 여름 Unity 2018.1 출시  
- .NET 4.x Equivalent가 Experimental에서 안정 버전으로
  
2018년 겨울 Unity 2018.3 출시  
- .NET 4.x Equivalent가 Default로, .NET 3.5 Equivalent는 비 추천으로
  
2019년 중 Unity 2019 출시 사이클 중  
- .NET 3.5 Equivalent가 이용 불가로 될 예정
  
<br/>  
  
Scripting Runtime Version을 바꾸면 Api Compatibility Level의 선택기가 바뀐다   
![](/images/2019/109.png)   
  
Api Compatibility Level을 바꾸면 프로젝트에서 참조하는 .NET의 DLL이 바뀐다
  
.NET 4.x를 선택한 경우  
- .NET Framework 4.x의 API 모두를 제공한다.
- API로서 제공되지만 모든 플랫폼에서 동작하는 것은 아니다. 실행 시 에러가 된다.
- 빌드 결과물의 크기가 커진다.
- 이것을 선택했을 때 .NET Standard 2.0 용 라이브러리도 사용할 수 있다.
  
  
.NET Standard 2.0을 선택한 경우  
- .NET 4.x에 제공된 API 중에서 이쪽에 없는 API도 있다.
- 빌드 결과물의 크기가 작아질 수 있다.
- 개발에 .NET Standard 2.0용 라이브러리를 사용할 수 있다.
  
  
Api Compatibility Level에서 .NET 4.x에서는 사용할 수 있지만 .NET Standard 2.0에서 사용할 수 없는 코드 예 `System.Reflection.Emit`  
  
  
.NET Standard는 .NET Framework, .NET Core, Mono의 “.NET 구현” 클래스 라이브러리가 제공해야할 API 기능, 규격.  
  
버전이 있으며, 버전 마다 API 개수가 다르다.  
- 1.0: 7949개의 API
- 1.6: 13501개의 API
- 2.0: 32638개의 API
  
Unity 2018.1 이후는 .NET Standard 2.0을 지원하고 있음.  
  
.NET Standard 1.0용 라이브러리를 사용할 수 있는 플랫폼  
Unity 2018.1, .NET Core 1.0, .NET Framework 4.5, Mono 4.6 이상  
  
.NET Standard 2.0용 라이브러리를 사용할 수 있는 플랫폼  
Unity 2018.1, .NET Core 2.0, .NET Framework 4.6.1, mono 5.4 이상  
  
  
.NET Standatd 버전 표  
![](/images/2019/110.png)   
  
  
현재 .NET Standard의 최신 버전은 .NET Standard 2.1  
