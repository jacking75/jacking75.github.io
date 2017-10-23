---
layout: post
title: Visual Studio - 참조 어셈블리와 NuGet 패키지의 타입의 using/Imports를 추가
published: true
categories: [Visual Studio]
tags: vs vc vs2017
---
인식되지 않는 타입이 입력된 경우 참조 어셈블리와 NuGet.org를 검색하여 using/Imports를 추가하도록 간단한 수정을 제안한다.   
이 기능은 default로는 무효로 되어 있다.  
유효화 하려면 [Tools]-[Options]-[Text Editor]-[C#] 또는 [Basic]-[Advanced] 순으로 진행하고, [Suggest usings for types in reference assemblies] 및 [Suggest usings for types in NuGet packages]을 선택한다.  
  
후자의 옵션을 선택하면 10 MB 정도의 NuGet 인덱스가 머신에 다운로드된다.  
다운로드에 몇 초 정도 걸리기 때문에 Visual Studio의 워크 플로에는 영향을 주지 않지만 이 기능을 유효화 되어도 즉석에서 사용 가능한 정도는 안 된다  

  
