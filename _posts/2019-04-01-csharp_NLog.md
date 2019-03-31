---
layout: post
title: C# - NLog
published: true
categories: [.NET]
tags: c# .net nlog log
---
## 특징
- 오픈소스 닷넷 플랫폼 로그 라이브러리
- .NET Compact, mono도 지원
- 사용이 쉽고, 설정은 설정 파일과 소스 코드를 통한 2가지 방법 제공.
- 옵션 지정으로 버퍼링, 비동기, 로드 밸런싱, 장애대처 등을 할 수 있다.
- 출력 옵션
    - Files - single file or multiple, with automatic file naming and archival
    - Event Log - local or remote
    - Database - store your logs in databases supported by .NET
    - Network - using TCP, UDP, SOAP, MSMQ protocols
    - Command-line console - including color coding of messages
    - E-mail - you can receive emails whenever application errors occur
    - ASP.NET trace 등등…
- NuGet 지원. [http://www.nuget.org/packages/NLog]
- 로그 뷰어. [https://github.com/nlog/nlog/wiki/Tools]
  
  
  
## 설치
- 가장 쉬운 방법은 위에 나왔듯이 Nuget으로 설치한다.
- 주의할 점은 Nuget에서 NLog뿐만이 아닌 'NLog Configuration'도 설치한다.
    - NLog Configuration는 NLog 설정 파일을(NLog.config) 설치해준다.
- NLog.config 파일에 로그 설정을 한다.
- File-Target 설명 [https://github.com/nlog/NLog/wiki/File-target]
  
  
  
## 기본 사용
- NLog.config 파일 설정 예
  
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <!--
  See https://github.com/nlog/nlog/wiki/Configuration-file
  for information on customizing logging rules and outputs.
   -->
  <targets>
    <target name="console" xsi:type="ColoredConsole" layout="${date:format=HH\:mm\:ss}| [TID:${threadid}] | ${stacktrace} | ${message}" />
    <target name="file" xsi:type="File" fileName="${basedir}/Logs/${date:format=MMddhhmmss}.log" layout="[${date}] [TID:${threadid}] [{stacktrace}]: ${message}" />
  </targets>

  <rules>
    <logger name="*" minlevel="Info" writeTo="file" />
    <logger name="*" minlevel="Info" writeTo="console" />
  </rules>
</nlog>
```
  
- 사용 코드
  
```
using NLog;

public class LoggerClass
{
  private static Logger Logger = LogManager.GetCurrentClassLogger();

  public static void NLogInfo(string message)
  {
      Logger.Info(message);
  }

  public static void NLogFatal(string message)
  {
      Logger.Fatal(message);
  }
}
```
  
  
  
## 예외 로그로 남기기
  
```
// 예외 발생
try {
    int i = 0;
    i = 1 / i;
}
catch (Exception ex)
{
    // 예외 로그 출력
    _logger.ErrorException("0 나누기 에러:" , ex);
}


<target name="file" xsi:type="File"
layout="${message} ${exception:format=tostring}"
fileName="${basedir}/log.txt" />
```
  
  
  
## 로그 설정 예제: 파일 크기 로테이션, 리치텍스트 컨트룰 출력, 특정 문자 포함 로그만 출력
  
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

<!--출력(Target) 설정-->
<targets>

    <!--파일 출력(10 KByte 마다 로테이션)-->
    <target name="file" xsi:type="File"
    layout="${longdate} ${level:uppercase=true:padding=-5} [${threadid}] ${logger} - ${message} ${exception:format=tostring}"
    fileName="${basedir}/logs/log.txt"
    archiveFileName="${basedir}/logs/${date:format=yyyyMMdd}/log.{###}.txt"  <!--로테이션으로 만들어진 로그 파일 이름 형식-->
    archiveAboveSize="10240"
    archiveNumbering="Sequence"
    maxArchiveFiles="20" />

    <!-- 폼의 리치 텍스트 컨트롤(색 입히기)-->
    <target name="rich" xsi:type="RichTextBox"
    formName="Form1"
    controlName="richTextBox1"
    useDefaultRowColoringRules="true" />

    <!--디버그/출력 윈도우-->
    <target name="debugger" xsi:type="Debugger"/>

    <!--「치명적」 이라는 문자를 포함 시에만 출력하는 필터링-->
    <target name="eventlog" xsi:type="FilteringWrapper"
        condition="contains('${message}','치명적')">
        <!--이벤트 로그-->
        <target xsi:type="EventLog"
            source="NLog Sample"
            log="Application" />
    </target>

</targets>

<!--출력 룰 설정-->
<rules>
    <logger name="Sample.*" minlevel="Info" writeTo="file,rich,debugger" />
    <logger name="*" levels="Error,Fatal" writeTo="eventlog" />
</rules>
</nlog>
```
  
  
  
## Logger 이름을 로그 출력 파일 이름으로 설정
- 로그 파일 이름이 현재 시간이 되도록 로거 이름을 현재 시간으로 설정
  
```
public static Logger Logger = LogManager.GetLogger(DateTime.Now.ToString());
</source>
<source lang="cpp">
<target name="file" xsi:type="File"
            fileName="${basedir}/Logs/${logger}.log"
            layout="[${date}] [TID:${threadid}] [${stacktrace}]: ${message}" />
 </targets>
```
  
  
  
##로그 파일 크기가 넘어서면 새로운 로그 파일 만들기
- 옵션 중 timeToSleepBetweenBatches의 기본은 50. 만약 0으로 설정하면 CPU 엄청 먹으므로 절대 0으로 하면 안된다
  
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <!--
  See https://github.com/nlog/nlog/wiki/Configuration-file
  for information on customizing logging rules and outputs.
   -->
  <targets>
    <target name="console" xsi:type="ColoredConsole" layout="${date:format=HH\:mm\:ss}| [TID:${threadid}] | ${stacktrace} | ${message}" />
    <target name="file" xsi:type="File"
            fileName="${basedir}/Logs/${logger}.log"
            archiveAboveSize="1000"
            archiveFileName="${basedir}/Logs/${date:format=MMddHHmm}/${shortdate}.{#####}.log"
            archiveNumbering="Sequence"
            layout="[${date}] [TID:${threadid}] [${stacktrace}]: ${message}" />
  </targets>

  <rules>
    <logger name="*" minlevel="Info" writeTo="file" />
    <logger name="*" minlevel="Info" writeTo="console" />
  </rules>
</nlog>
```
  
  
  
## 비동기로 로그 남기기
  
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <!--
  See https://github.com/nlog/nlog/wiki/Configuration-file
  for information on customizing logging rules and outputs.
   -->
  <targets>
    <!--<target name="console" xsi:type="ColoredConsole" layout="${date:format=HH\:mm\:ss}| [TID:${threadid}] | ${message}" />-->

    <target name="AsyncConsoleLog"
            xsi:type="AsyncWrapper" overflowAction="Block" queueLimit="1000000" batchSize="1000" timeToSleepBetweenBatches="1">
      <target name="console" xsi:type="ColoredConsole" layout="${date:format=HH\:mm\:ss}| [TID:${threadid}] | ${message}" />
    </target>

    <target name="AsyncInfoLog"
            xsi:type="AsyncWrapper" overflowAction="Block" queueLimit="1000000" batchSize="1000" timeToSleepBetweenBatches="1">
      <target name="InfoFile" xsi:type="File"
            fileName="${basedir}/Logs/Info_${logger}.log"
            archiveAboveSize="128000000"
            archiveFileName="${basedir}/Logs/Info_${date:format=MMddHHmm}/${shortdate}.{#####}.log"
            archiveNumbering="Sequence"
            concurrentWrites="false"
            layout="[${date}] [TID:${threadid}] [${stacktrace}] ${message}" />
    </target>

    <target name="AsyncErrorLog"
            xsi:type="AsyncWrapper" overflowAction="Block" queueLimit="1000000" batchSize="1000" timeToSleepBetweenBatches="1">
      <target name="ErrorFile" xsi:type="File"
            fileName="${basedir}/Logs/Error_${logger}.log"
            archiveAboveSize="128000000"
            archiveFileName="${basedir}/Logs/Error_${date:format=MMddHHmm}/${shortdate}.{#####}.log"
            archiveNumbering="Sequence"
            concurrentWrites="false"
            layout="[${date}] [TID:${threadid}] [${stacktrace}] ${message}" />
    </target>

  </targets>

  <rules>
    <logger name="*" minlevel="Info" maxlevel="Info" writeTo="AsyncInfoLog" />
    <logger name="*" minlevel="Error" writeTo="AsyncErrorLog" />

    <!-- 서비스 시에는 콘솔은 주석 처리하자 -->
    <logger name="*" minlevel="Info" writeTo="AsyncConsoleLog" />
  </rules>
</nlog>
```
  
  
  
## csv 포맷으로 출력하기
  
```
<target name="AsyncInfoLog"
        xsi:type="AsyncWrapper" overflowAction="Block" queueLimit="1000000" batchSize="1000" timeToSleepBetweenBatches="1">
  <target name="InfoFile" xsi:type="File"
        fileName="${basedir}/Logs/Info_${logger}.log"
        archiveAboveSize="128000000"
        archiveFileName="${basedir}/Logs/Info_${date:format=MMddHHmm}/${shortdate}.{#####}.log"
        archiveNumbering="Sequence"
        concurrentWrites="false">
        <layout xsi:type="CsvLayout">
          <column name="time" layout="${date}" />
          <column name="tid" layout="${threadid}"/>
          <column name="stack" layout="${stacktrace}"/>
          <column name="message" layout="${message}" />
        </layout>
  </target>
</target>
```
  
<pre>
time,tid,stack,message
2015/07/06 15:27:40.398,1,Program.Main => FileLogger.Info,[24]서버 시작 준비 ~~~


메시지 안에서 , 을 사용한 경우 메시지를 "로 묶는다
2015/07/06 15:27:43.428,1,Program.Main => FileLogger.Info,"[47]최소 워커 스레드 수: 8, 최소 IO 스레드 수:8 | 최대 워커 스레드 수: 32767, 최대 IO 스레드 수:1000"
</pre>
  
  
  
## 릴리즈 빌드에서는 로그를 완전 제거
- Logger.ConditionalTrace("entering method {0}", methodname);
  
  
  
## 참고
- 본인이 만든 문서 [http://www.slideshare.net/jacking/n-log]
- NLog 관련 글 모음 [https://github.com/nlog/nlog/wiki/Web-resources]
- [(일어)NLog 자주 사용하는 레이아웃](http://qiita.com/shusakuorz/items/47be13a66a7c93cef460)
- [(일어)NLog 출력하는 파일을 나누기](http://qiita.com/shusakuorz/items/33b391968b5f11019f69)
- How to NLog (2.1 & 3.1) with VisualStudio 2013 [http://www.codeproject.com/Tips/749612/How-to-NLog-with-VisualStudio]
- .NET low latency logging. Part 1 - Testing environment, Sample application, Best performance! [http://deep-depth.blogspot.kr/2014/01/choose-solution-for-low-latency-logging.html]
- .NET low latency logging. Part 2 - NLog performance [http://deep-depth.blogspot.kr/2014/01/net-low-latency-logging-part-2-nlog.html]
- .NET low latency logging. Part 3 - log4net performance [http://deep-depth.blogspot.kr/2014/01/net-low-latency-logging-part-3-log4net.html]
