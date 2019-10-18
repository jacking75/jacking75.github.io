---
layout: post
title: C# - 잡지 못한 예외 처리
published: true
categories: [.NET]
tags: .net c# .netcore exception
---
## Application.ThreadException 이벤트 사용
Windows Form 애플리케이션에서는 잡지 못한 예외가 throw 되면 Application.ThreadException 이벤트가 발생한다.  
단 ThreadException 이벤트가 발행할 수 있는 부분은 Windows Form 작성, 소유하고 있는 스레드(UI 스레드)에서 예외가 throw 될 때이다.  
Thread.Start 메소드나 BeginInvoke 메소드 등에서 시작된 스레드에서 발생한 예외에서는 이벤트가 발생하지 않는다.  
ThreadException 아벤트 핸들러가 호출될 때 아무것도 하지 않으면 예외는 무시된다(즉 애플리케이션은 종료되지 않는다).  
  
![](/images/2019/106.png) 
  
  
## ThreadException 이벤트 발생 옵션
기본적으로 ThreadException 이벤트 발생 여부는 애플리케이션 구성 파일 설정에 따른다.  
애플리케이션 구성 파일 설정에 관계 없이 항상 ThreadException이벤트가 발생되도록 하려면 Application.SetUnhandledExceptionMode 메서드에 UnhandledExceptionMode.CatchException을 지정한다.  
만약 항상 ThreadException이벤트를 발생하지 않으려면 UnhandledExceptionMode.ThrowException을 지정한다.  
SetUnhandledExceptionMode 메서드는 Application.Run 전에 호출해야 한다.  
SetUnhandledExceptionMode 메서드는 2번째 파라미터에 False를 지정하여 애플리케이션 예외 모드로 한다(기본 값은 스레드 예외 모드).   
  
![](/images/2019/107.png) 
  
  
## AppDomain.UnhandledException 이벤트 사용
AppDomain.UnhandledException이벤트를 사용해서 잡지 못한 예외를 처리할 수 있다.  
UnhandledException 이벤트는 ThreadException 이벤트와 달리 UI 스레드 이외의 스레드에서 예외가 throw 되는 경우라도 발생한다. 또 Windows Form 애플리케이션만 아니라 콘솔 응용 프로그램에서도 사용할 수 있다.  
  
또 ThreadException 이벤트와 달리 UnhandledException 이벤트 핸들러가 호출된 뒤에도 예외를 무시하지 않아서 그대로 두면 통상은 "(애플리케이션)는 동작을 정지했습니다." 라는 대화 상자가 표시되면서 애플리케이션은 종료한다.  
UnhandledException 이벤트 핸들러에서 애플리케이션을 종료시킬 때 Environment.Exit가 아니라 Application.Exit메서드를 사용하면 "(애플리케이션)는 동작을 정지했습니다.“ 라는 대화 상자가 표시된다  
  
![](/images/2019/108.png) 
  
  
  
## 잡지 못하는 예외
앞의 두 이벤트로도 예외를 잡지지 못하는 경우가 있다.  
.NET Framework 4.0에서는 기본적으로 파손 상태 예외(CSE)를 잡을 수 없다.  
자세한 것은 "HandleProcessCorruptedStateExceptionsAttribute 클래스"나 "파손 상태 예외를 처리", [All about Corrupted State Exceptions in . NET4](http://dotnetslackers.com/articles/net/All-about-Corrupted-State-Exceptions-in-NET4.aspx ) 등을 참고해라.  
  
64비트 환경에서는 Form의 Load 이벤트 핸들러에서 throw된 예외가 무시된다는 문제가 있다.  
[c#-VS2010 does not show unhandled exception message in a WinForms Application on a 64-bit version of Windows](http://stackoverflow.com/questions/4933958/vs2010-does-not-show-unhandled-exception-message-in-a-winforms-application-on-a/4934010#4934010 ) 등을 참고해라.  