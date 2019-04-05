---
layout: post
title: C# - config-r
published: true
categories: [.NET]
tags: c# .net config-r
---
## 개요
- scripts와 Roslyn을 사용하여 스크립트 방식의 설정 파일을 사용할 수 있다.
- 저장소 https://github.com/config-r/config-r
- NuGet으로 설치
  
  
  
##  Quickstart
1. NuGet으로 설치
2. 실행 파일과 같은 이름의 .csx 파일을 프로젝트에 추가(예 실행파일 이름이 ConsoleApplication1.exe 라면 ConsoleApplication1.exe.csx). 추가한 파일의 프로젝트 속성 'Copy to Output Directory'에서 Web 프로젝트는 복사 안함, 그 외 프로젝트는 항상 복사로 설정한다.
  
```
// ConsoleApplication1.exe.csx
// regular C#, no restrictions!*
Add("Count", 123);
Add("Uri", new Uri("https://github.com/config-r/config-r"));
```
  
```
// program.cs
void Main(string[] args)
{
    var count = Config.Global.Get<int>("Count");     // it's a System.Int32!
    var uri = Config.Global.Get<Uri>("Uri");         // it's a System.Uri!
    Console.WriteLine("Count: {0}", count);
    Console.WriteLine("Uri: {0}", uri);
}
```
  
config-r을 라이브러리 프로젝트에서 설치했다면 .csx 파일을 실행 파일 프로젝트에 추가하면 된다.
  
  
  
## 설정 파일 위치 변경
- 기본적인 방법
  
```
Config.Global.LoadScriptFile("path/to/MyConfig.csx");
```
  
- 웹상에 있는 설정 파일 사용
  
```
Config.Global.LoadWebScript(new Uri(
    "https://gist.github.com/adamralph/6843899/raw/8cfdb09ad00655edb389cb0761aca44fb24f83fb/sample-config2.csx"));
```
  
- 복수의 파일 로딩
  
```
Config.Global.LoadScriptFile("Custom1.csx").LoadScriptFile("Custom2.csx");
```
  
- 로딩한 스크립트 파일 언로드
  
```
Config.Global.Unload();
```
  
  
  
## 동적으로 어셈블리 로딩
  
```
// MyApp.exe
var assembly = typeof(Foo).Assembly; // Alternatively Assembly.Load("Path to assembly");
Config.GlobalAutoLoadingReferences.Add(assembly);
var myConfigItem = Config.Global.Get<object>("myconfigitem");
```
  
```
// MyApp.csx
Add("myconfigitem", new SomeTypeFromTheFooAssembly());
```
  
스크립트 파일의 위치를 지정
  
```
// MyApp.exe
var assembly = typeof(Foo).Assembly;
Config.Global.LoadScriptFile("\\path\to\MyConfig.csx", assembly);
var myConfigItem = Config.Global.Get<object>("myconfigitem");
```
  
  
  
## 기능
### 원격에 있는 스크립트 실행
  
```
Load("\\configserver\configshare\{app name}\{machine name}.csx");
```
  
```
var foo = Configurator.Get<int>("foo");
```
  
  
### key-value
  
```
Add("The Answer to Life, The Universe, and Everything", 42);

var answer = Config.Global.Get<int>("The Answer to Life, The Universe, and Everything");
```
  
  
### 다른 파일에서 데이터 로딩
Custom2.Data.csx
  
```
var count = 345;
var uri = new Uri("https://github.com/config-r/config-r/foobarbazfoobarbaz");
var fromCustom1File = false;
var fromCustom2File = true;
```
  
Custom2.csx
  
```
#load "Custom2.Data.csx"

Add("Count", count);
Add("Uri", uri);
Add("FromCustom1File", fromCustom1File);
Add("FromCustom2File", fromCustom2File);

Load("Custom3.csx");
```
  
Custom3.csx
  
```
Add("Count", 456);
Add("Uri", new Uri("https://github.com/config-r/config-r/foobarbaz"));
Add("FromCustom1File", false);
Add("FromCustom2File", false);
Add("FromCustom3File", true);
```
  
  
### 호스트 프로젝트에서 정의한 클래스 생성
  
```
#r "ConfigR.Samples.ConsoleApplication.exe"

using ConfigR.Samples.ConsoleApplication;

Add("Foo", new Foo { Bar = "Baz" }); // the Foo class is defined in the applicaton!
```
  
  
### 호스트에서 생성한 객체 사용하기
  
```
Add("Bar", Configurator.Get<int>("Foo") * 2); // the 'Foo' value was added in the application!
```
  
  
### 스크립트로 작업 스케쥴링 하기
  
```
#r "ConfigR.Samples.Scheduler.exe"
#r "Common.Logging.Core.dll"

using System;
using Common.Logging;
using ConfigR.Samples.Scheduler;

Add(
    "Schedules",
    new[]
    {
        new Schedule
        {
            Action = () =>
                {
                    LogManager.GetCurrentClassLogger().Info("The 1st schedule is sending some emails!");
                    // send some emails
                },
            NextRun = DateTime.Now.AddSeconds(2),
            RepeatInterval = TimeSpan.FromSeconds(4),
        },
        new Schedule
        {
            Action = () =>
                {
                    LogManager.GetCurrentClassLogger().Info("The 2nd schedule is downloading some reports!");
                    // download some reports
                },
            NextRun = DateTime.Now.AddSeconds(3),
            RepeatInterval = TimeSpan.FromSeconds(4),
        },
        new Schedule
        {
            Action = () =>
                {
                    LogManager.GetCurrentClassLogger().Info("The 3rd schedule is performing some housekeeping!");
                    // perform some housekeeping
                },
            NextRun = DateTime.Now.AddSeconds(4),
            RepeatInterval = TimeSpan.FromSeconds(4),
        }
    });
```
  