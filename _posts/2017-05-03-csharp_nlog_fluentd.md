---
layout: post
title: NLog - Fluentd 노드로 로그를 보내는 방법
published: true
categories: [.NET]
tags: c# .net nlog fluentd
---
NuGet에서 아래 2개를 설치한다  
- [NLog](https://www.nuget.org/packages/NLog)
- [NLog.Targets.Fluentd](https://www.nuget.org/packages/NLog.Targets.Fluentd/)  
  
Fluentd 노드로 로그를 보내기 위한 설정은 NLog.config에서 지정할 수 없으므로 코드 상에서 설정해야 한다.    
      
```
public class Configurator
{
    // Fluentd 설정을 반환한다
    static NLog.Targets.Target GetFluentdTarget()
    {
        var target = new NLog.Targets.Fluentd();
        target.Layout = new NLog.Layouts.SimpleLayout("${message}");
        target.Host = "１２７．０．０．１"; // TODO: 상황에 맞게 변경한다
        target.Port = 24224; // TODO: 상황에 맞게 변경한다
        target.Tag = "tag.tag"; // TODO: 상황에 맞게 변경한다
        target.NoDelay = true;
        target.LingerEnabled = false;
        target.LingerTime = 2;
        target.EmitStackTraceWhenAvailable = false;
        return WrapTarget(target);
    }

    // Fluentd 설정만으로는 동기적으로 로그를 보내므로 비동기&리트라이 설정을 한다.
    static NLog.Targets.Target WrapTarget(NLog.Targets.Target target)
    {
        var retryWrapper = new NLog.Targets.Wrappers.RetryingTargetWrapper("RetryingWrapper", target, 3, 1000);
        var asyncWrapper = new NLog.Targets.Wrappers.AsyncTargetWrapper("AsyncWrapper", retryWrapper);
        return asyncWrapper;
    }

    public static NLog.Config.LoggingConfiguration GetConfig()
    {
        var config = new NLog.Config.LoggingConfiguration();
        var target = GetFluentdTarget();
        config.AddTarget("fluentd", target);
        config.LoggingRules.Add(new NLog.Config.LoggingRule("*", LogLevel.Debug, target));
        return config;
    }
}

// Global.asax.cs(애플리케이션 초기화 시에 설정한다)
public class Global : HttpApplication
{
    protected void Application_Start()
    {
        // Fluentd 설정을 하는 Config를 설정한다
        LogManager.Configuration = LogConfigurator.GetConfig();
    }
}
```
  
    
<br>
<br>  

출처: http://qiita.com/akehoyayoi@github/items/87b268327dd2c2b6cd73