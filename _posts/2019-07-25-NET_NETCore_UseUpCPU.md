---
layout: post
title: CPU 사용률이 오르지 않을 때
published: true
categories: [.NET]
tags: .net c# asp.net webapi
---
처리하는 것에 비해 CPU 사용률이 오르지 않는 경우 .NET 런타임의 GC 설정을 변경해서 해결 할 수도 있다.  
.NET의 GC 모드를 서버GC로 변경한다.  
  
## .NET Core
프로젝트 파일에 ServerGarbageCollection을 추가하고 값을 true로 한다.  
```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <ServerGarbageCollection>true</ServerGarbageCollection>
  </PropertyGroup>

  ...

</Project>
```
  
  
## .NET Framework
App.config에 gcServer 라는 요소를 추가하고 enabled 속성 값을 true로 한다.  
```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup> 
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6.1" />
  </startup>
  <runtime>
    <gcServer enabled="true"/>
  </runtime>
</configuration>
```  