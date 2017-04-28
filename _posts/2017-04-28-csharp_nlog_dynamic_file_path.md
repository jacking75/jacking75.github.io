---
layout: post
title: NLog - 기존의 설정을 사용하면서 파일 패스만 동적으로 변경하기
published: true
categories: [.NET]
tags: c# .net nlog
---
NLog.config 설정을 사용하지만 파일 패스만 바꾸고 싶다. 파일 경로는 프로세스나 인스턴스 마다 변경하기를 바란다.  
  
프로세스나 인스턴스가 항상 일정하다면 NLog.config에 필요한 수만큼의 설정을 추가하면 되지만, 파일 패스만 다르므로  비슷한 설정 코드를 늘리는 것은 귀찮다. 
  
그래서 로거마다 Variables 속성 값을 독립적으로 설정하는 방법으로 하였다.  
    
NLog.config
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <targets>
  <target
    name="LogFile"
    xsi:type="File"
    layout="${longdate} [${uppercase:${level:padding=-5}}] ${message} ${exception:format=tostring}"
    fileName="${basedir}Logs\${var:runtime}\${date:format=yyyyMMdd}.log"
    encoding="UTF-8"
    archiveFileName="${basedir}Logs\archives\${var:runtime}\archive.{#}.log"
    archiveEvery="Day"
    archiveNumbering="Rolling"
    maxArchiveFiles="7"
    header="[Start Logging]"
    footer="[End Logging]${newline}"  />
  </targets>

  <rules>
    <logger name="Log" minlevel="Trace" writeTo="LogFile" />
  </rules>
</nlog>
```
  
아래는 Nlog를 사용하는 코드 부분이다.  
  
```
var factory = new LogFactory();
var logger = factory.GetLogger("Log");
logger.Factory.Configuration.Variables.Add("runtime", "test");
factory.ReconfigExistingLoggers();
``` 
  
LogFactory 인스턴스를 직접 생성하고 LogFactory.GetLogger 메서드를 호출하고 있다.  
logger.Factory 속성을 보면 알겠지만 이 속성은 LogFactory.GetLogger 메서드로 취득한 경우, 항상 동일한 값을 가리키기 때문에 당연히 Configuration.Variables 속성의 내용도 일치한다.
  
이렇게하면 아래와 같이 Logger 인스턴스마다 출력처를 쉽게 바꿀 수 있다.  
  
```
var factory1 = new LogFactory();
var logger1 = factory1.GetLogger("Log");
logger1.Factory.Configuration.Variables.Add("runtime", "test1");
factory1.ReconfigExistingLoggers();

var factory2 = new LogFactory();
var logger2 = factory2.GetLogger("Log");
logger2.Factory.Configuration.Variables.Add("runtime", "test2");
factory2.ReconfigExistingLoggers();

logger1.Warn("message"); // test1 폴더 아래에 출력
logger2.Warn("message"); // test2 폴더 아래에 출력
```  
  
<br>
<br>  

출처: http://ouranos.sakura.ne.jp/wordpress/2017/02/27/%E9%96%8B%E7%99%BA%E3%83%A1%E3%83%A2-%E3%81%9D%E3%81%AE32-nlog%E3%81%A7%E6%97%A2%E5%AD%98%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%82%92%E6%B5%81%E7%94%A8%E3%81%97%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%83%91/