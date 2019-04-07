---
layout: post
title: C# - serilog
published: true
categories: [.NET]
tags: c# .net serilog log
---
- .NET의 많은 다른 라이브러리와 마찬가지로 Serilog는 파일, 콘솔 등의 다양한 저장소로의 로깅을 제공한다. 또한 설정하기 쉬운 깨끗한 API를 가지고 있으며, 다양한.NET 플랫폼으로 이식성을 가지고 있다. 또 다른 로깅 라이브러리와는 달리 Serilog는 강력한 구조화 된 이벤트 데이터로 구축된다.
- http://serilog.net/
- NLog와 연동될 수도 있지만 별로 매끄럽게 되지 않는다. NLog에서 정의한 설정 정보를 사용하지만 완벽하지 않은듯하다.
    - NLog 설정에서 정한 이름과 다름 이름으로 로그 파일이 생성된다.
    - NLog에서 파일 로테이션을 파일 사이즈로 정해도 제대로 적용되지 않는다.
- 비동기 쓰기를 지원하지 않는다.
    - 비동기로 사용하고 싶다면 Task.Run을 사용해야 한다.
  
```
var position = new { Latitude = 25, Longitude = 134 };
var elapsedMs = 34;

log.Information("Processed {@Position} in {Elapsed:000} ms.", position, elapsedMs);

//09:14:22 [Information] Processed { Latitude: 25, Longitude: 134 } in 034 ms.
```
  
  
  
## 설정
- Nuget으로 손 쉽게 설치 할 수 있다.
- Settup
  
```
using Serilog;

var log = new LoggerConfiguration()
    .WriteTo.ColoredConsole()
    .CreateLogger();

log.Information("Hello, Serilog!");
```
  
- 전역 로그 설정. static 변수를 선언하거나 Log.Logger을 사용한다.
  
```
Log.Logger = log;
Log.Information("The global logger has been configured");
```
  
  
  
## Sinks(로그 출력처)
- WriteTo를 사용하여 sinks를 설정한다.
  
```
Log.Logger = new LoggerConfiguration()
    .WriteTo.ColoredConsole()
    .CreateLogger();

Log.Information("Ah, there you are!");
```
  
- 복수 sinks 설정도 가능하다
  
```
Log.Logger = new LoggerConfiguration()
    .WriteTo.ColoredConsole()
    .WriteTo.RollingFile(@"C:\Log-{Date}.txt")
    .CreateLogger();
```
  
- 로그 레벨 설정
  
```
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.ColoredConsole()
    .CreateLogger();
```
  
디버깅 이상 레벨만 출력한다.  
디버깅 레벨을 설정하지 않으면 default는 Info 이다.  
  
- sinks마다 로그 레벨 다르게 설정
  
```
Log.Logger = new LoggerConfiguration()
    .WriteTo.ColoredConsole()
    .WriteTo.MongoDB("mongo://myserver/logs", minimumLevel: LogEventLevel.Warning)
    .CreateLogger();
```
콘솔 로그는 Info 이상, MongoDB는 Wraning 이상  
  
- Full .NET Framework
Azure Table Storage, Colored Console, Console, CouchDB, Dump File, ElasticSearch, elmah.io, Email, File, Glimpse, log4net, Logentries, Loggly, Loggr, MongoDB, MS SQL Server, NLog, RavenDB, Rolling File, Seq, Splunk, Trace, Windows Event Log  
    - https://github.com/serilog/serilog/wiki/Provided-Sinks
  
  
  
## Enrichers
로그 포맷에 사용되는 속성을 추가 및 삭제 할 수 있다.  
```
class ThreadIdEnricher : ILogEventEnricher
{
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        logEvent.AddPropertyIfAbsent(
            propertyFactory.CreateProperty("ThreadId", Thread.CurrentThread.ManagedThreadId));
    }
}
```
  
위의 속성을 아래에서 사용하도록 설정한다.  
```
Log.Logger = new LoggerConfiguration()
    .Enrich.With(new ThreadIdEnricher())
    .WriteTo.ColoredConsole(
        outputTemplate: "{Timestamp:HH:mm} [{Level}] ({ThreadId}) {Message}{NewLine}{Exception}")
    .CreateLogger();
```
  
아래처럼 구성을 간소화 할 수 있다.  
```
Log.Logger = new LoggerConfiguration()
    .Enrich.WithProperty("Version", "1.0.0")
    .WriteTo.ColoredConsole()
    .CreateLogger();
```
  
  
  
## Filters
로그 문맥을 분석해서 필터링 할 수 있다.  
```
Log.Logger = new LoggerConfiguration()
    .WriteTo.ColoredConsole()
    .Filter.ByExcluding(Matching.WithProperty<int>("Count", p => p < 10))
    .CreateLogger();
```
  
위 코드는 Count의 값이 10 보다 작을 때만 로그를 남긴다.
  
  
  
## Writing Log Events
quota는 int, user은 string 이다.  
```
Log.Warning("Disk quota {Quota} MB exceeded by {User}", quota, user);
//Disk quota 1024 MB exceeded by "nblumhardt"
```
  
string은 ""가 같이 출력되는데 제거하고 싶다면 {User:l} 한다.  
```
Log.Information("The time is " + DateTime.Now);
```
  
보다는 아래 방식이 좋다.  
```
Log.Information("The time is {Now}", DateTime.Now);
```
  
  
  
## 로그 이벤트 레벨
- 레벨 순서
  
```
Verbose < Debug < Information  < Warning < Error < Fatal 
```
  
- 레벨 조사
  
```
readonly bool _isDebug = Log.IsEnabled(LogEventLevel.Debug);
```
  
- 동적 로그 레벨 설정
  
```
var levelSwitch = new LoggingLevelSwitch();

var log = new LoggerConfiguration()
  .MinimumLevel.ControlledBy(levelSwitch)
  .WriteTo.ColoredConsole()
  .CreateLogger();

levelSwitch.MinimumLevel = LogEventLevel.Verbose;
log.Verbose("This will now be logged");
```
  
  
  
## Structured Data
- Simple, Scalar Values
  
```
var count = 456;
Log.Information("Retrieved {Count} records", count);

//{ "Count": 456 }
```
  
- Collections
  
```
var fruit = new[] { "Apple", "Pear", "Orange" };
Log.Information("In my bowl I have {Fruit}", fruit);

//{ "Fruit": ["Apple", "Pear", "Orange"] }
```
  
```
var fruit = new Dictionary<string,int> {{ "Apple", 1}, { "Pear", 5 }};
Log.Information("In my bowl I have {Fruit}", fruit);

//{ "Fruit": { "Apple": 1, "Pear": 5 }}
```
  
- Objects
  
```
SqlConnection conn = ...;
Log.Information("Connected to {Connection}", conn);
```
  
conn에 ToString()이 정의 되어 있어야 한다.  
```
var sensorInput = new { Latitude = 25, Longitude = 134 };
Log.Information("Processing {@SensorInput}", sensorInput);
```
  
  
  
## Rolling File
- 파일 이름에 {Date}을 붙임.
  
```
var log = new LoggerConfiguration()
    .WriteTo.RollingFile("log-{Date}.txt")
    .CreateLogger();
```
  
- default로 로그 파일의 크기는 1GB. 변경도 가능.
```
.WriteTo.RollingFile("log-{Date}.txt", fileSizeLimitBytes: null)
```
  
- 이미 만들어진 로그 파일은 default로 31개까지만 저장. 변경 가능
```
.WriteTo.RollingFile("log-{Date}.txt", retainedFileCountLimit: null)
```
  
  
  
## Samples
- https://github.com/serilog/serilog-samples
  



