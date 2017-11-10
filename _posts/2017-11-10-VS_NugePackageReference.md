---
layout: post
title: Visual Studio - Nuget 패키징 포맷이 packages.config 뿐만이 아닌 PackageReference도 사용 가능
published: true
categories: [Visual Studio]
tags: vs c# vs2017
---
Nuget으로 라이브러리를 설치하면 기존까지는 packages.config 파일이 만들어지면 이 파일에 관련 정보가 들어가 있었다.  
그러나 VS2017부터는 .NetCore에서 사용 하는 방식처럼 PackageReference도 사용할 수 있다.  
PackageReference를 사용하면 프로젝트 설정 파일에 nuget 정보가 들어간다.  
PackageReference를 사용하려면 아래 화면의 옵션 화면에서 사용하도록 선택해야 한다.    
      
![](/images/2017/10-19_001.PNG)  
    
  
