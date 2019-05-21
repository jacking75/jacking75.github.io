---
layout: post
title: C# - 예외 출력 helper 클래스
published: true
categories: [.NET]
tags: c# .net Exception
---
출처: https://teratail.com/questions/24669
   
```
public static class ExceptionHelper
{
    public static string ExtractException(this Exception ex, int indent = 2)
    {
        var indentStr = new String(' ',indent);
        StringBuilder traceLog = new StringBuilder();
        StackTrace trace = new StackTrace(ex, true);
        foreach (var frame in trace.GetFrames())
        {
            traceLog.AppendLine($"{indentStr}File Name : {frame.GetFileName()}");
            traceLog.AppendLine($"{indentStr}Class Name : {frame.GetMethod().ReflectedType.Name}");
            traceLog.AppendLine($"{indentStr}Method Name : {frame.GetMethod()}");
            traceLog.AppendLine($"{indentStr}Line Number : {frame.GetFileLineNumber()}");
            traceLog.AppendLine($"=======================================================");
        }

        return traceLog.ToString();
    }
}
```
  