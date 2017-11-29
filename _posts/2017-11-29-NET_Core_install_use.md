---
layout: post
title: .NET Core 설치 및 프로젝트 생성하기
published: true
categories: [.NET]
tags: .net c# .netcore
---
[원문](https://qiita.com/kumasan1011/items/49748c5587a7d9d0f9f1)  
  
## 설치
- Windwos
    - 다운로드 후 설치
    - https://www.microsoft.com/net/core#windowscmd
- Ubuntu
    - 패키지 추가
    - curl https://packages.microsoft.com/keys/microsoft.asc| gpg-dearmor>microsoft.gpg
    - sudo mv microsoft.gpg/etc/apt/trusted.gpg.d/microsoft.gpg
    - 설치는 Ubuntu 버전마다 좀 다르다
    - Ubuntu 17.04
    - sudo sh-c'echo"deb[arch=amd64]https://packages.microsoft.com/repos/microsoft-ubuntu-zesty-prod zesty main">/etc/apt/sources.list.d/dotnetdev.list'
    - Ubuntu 16.04
    - sudo sh-c'echo"deb[arch=amd64]https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main">/etc/apt/sources.list.d/dotnetdev.list'
    - Ubuntu 14.04
    - sudo sh-c'echo"deb[arch=amd64]https://packages.microsoft.com/repos/microsoft-ubuntu-trusty-prod trusty main">/etc/apt/sources.list.d/dotnetdev.list'
    - 다음은 항상 그렇듯
    - sudo apt-get update
    - sudo apt-get install dotnet-sdk-2.0.0
  
  
## 신규 프로젝트/파일의 작성
- Console Application
    - dotnet new console-o hogehoge
- Class library
    - dotnet new classlib-o hogehoge
- Unit Test Project
    - dotnet new mstest-o hogehoge
- xUnit Test Project
    - dotnet new xunit-o hogehoge
- ASP.NET Core Empty
    - dotnet new web-o hogehoge
- ASP.NET Core Web App(Model-View-Controller)
    - dotnet new mvc-o hogehoge
- ASP.NET Core Web App
    - dotnet new razor-o hogehoge
- ASP.NET Core with Angular
    - dotnet new angular-o hogehoge
- ASP.NET Core with React.js
    - dotnet new react-o hogehoge
- ASP.NET Core with React.js and Redux
    - dotnet new reactredux-o hogehoge
- ASP.NET Core Web API
    - dotnet new webapi-o hogehoge
- global.json file
    - dotnet new globaljson-o hogehoge
- Nuget Config
    - dotnet new nugetconfig-o hogehoge
- Web Config
    - dotnet new webconfig-o hogehoge
- Solution File
    - dotnet new sln-o hogehoge
- Razor Page
    - dotnet new page-o hogehoge
- MVC ViewImports
    - dotnet new viewimports-o hogehoge
- MVC ViewStart
    - dotnet new viewstart-o hogehoge
- 패키지 추가(Nuget 등)
    - dotnet add package hogehoge
  
  
## 의존 파일 복원
- Config 파일을 다시 쓴 경우 등
    - dotnet restore
  
   
## 실행
dotnet run  
  
  
  



