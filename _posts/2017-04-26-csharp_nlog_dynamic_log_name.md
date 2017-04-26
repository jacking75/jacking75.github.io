---
layout: post
title: NLog - 동적으로 로그 파일 이름 설정하기
published: true
categories: [.NET]
tags: c# .net nlog
---
NLog는 동적으로 파일 이름을 설정할 수 있는 기능이 있다.
  
아래의 방법은 커스텀 플레이스 홀더를 설정하고, 그 플레이스 홀더에 임의의 텍스트를 설정하는 방법으로 파일 이름을 변경한다.  
  
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
  
위의 *${var:runtime}*에 주목하자.  
이 부분이 프로그램 실행 중에 변화하는 부분이다. runtime 부분을 임의의 문자열로 설정할 수 있다. 
  
아래는 Nlog를 사용하는 코드 부분이다.  

```
var logger = LogManager.GetLogger("Log");
logger.Factory.Configuration.Variables.Add("runtime", "test");
logger.Factory.ReconfigExistingLoggers();
``` 
  
Variables 속성에 아까의 runtime을 키로 임의의 값을 설정하고 있다.
그리고 설정을 반영하기 위하여 ReconfigExistingLoggers 메서드를 호출한다.

이것으로 출력지의 실제 패스가 실행 폴더 경로\Logs\test\yyyyMMdd.log 라는 로그로 해석된다.  
  
<br>
<br>  
  
  
출처: http://ouranos.sakura.ne.jp/wordpress/2017/02/26/%E9%96%8B%E7%99%BA%E3%83%A1%E3%83%A2-%E3%81%9D%E3%81%AE31-nlog%E3%81%A7%E5%8B%95%E7%9A%84%E3%81%AB%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E5%90%8D%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B/