---
layout: post
title: C# - Windows Event Log
published: true
categories: [.NET]
tags: c# .net eventlog
---
- 개발 시에 Windows가 제공하는 이벤트 로그를 사용하면 로그 보기/관리가 쉬워진다.
- C# 을 이용해 Windows Event Log 쓰기 [http://mainia.tistory.com/412]
  
```
static void Main(string[] args)
{
    WriteEventLogEntry("This is an entry in the event log by daveoncsharp.com");
}

private static void WriteEventLogEntry(string message)
{
    // Create an instance of EventLog
    System.Diagnostics.EventLog eventLog = new System.Diagnostics.EventLog();

    // Check if the event source exists. If not create it.
    if (!System.Diagnostics.EventLog.SourceExists("TestApplication"))
    {
        // "Application" 이름이 로그 트리에 나오므로 잘 정의해야 한다.
        System.Diagnostics.EventLog.CreateEventSource("TestApplication", "Application");
    }

    // Set the source name for writing log entries.
    eventLog.Source = "TestApplication";

    // Create an event ID to add to the event log
    int eventID = 8;

    // Write an entry to the event log.
    eventLog.WriteEntry(message,
                        System.Diagnostics.EventLogEntryType.Error,
                        eventID);

    // Close the Event Log
    eventLog.Close();
}
```
  
- 이벤트 로그 보기
    - 제어판 -> 관리 도구 -> 이벤트 뷰어
- 이벤트 로그를 C#에서 읽기
    - 출처: http://social.msdn.microsoft.com/Forums/en-US/5bdd8518-5509-4414-a6a3-aa2e31c1bcc1/reading-event-logs-in-c-increasing-performance?forum=csharplanguage
  
```
using System;
using System.Diagnostics;
using System.Security;
using System.Text;
using System.Collections.Generic;

namespace glog
{
  class Program
  {
    static void Main(string[] args)
    {
      String machine = "."; // local machine
      Console.WriteLine("-------------------------------------------------------------\n");
      if (args.Length == 1)
      {
        if (args[0] == "application" || args[0] == "system" || args[0] == "security")
        {
          String log = args[0];
          EventLog aLog = new EventLog(log, machine);
          EventLogEntry entry;
          EventLogEntryCollection entries = aLog.Entries;
          Stack<EventLogEntry> stack = new Stack<EventLogEntry>();
          for (int i = 0; i < entries.Count; i++)
          {
            entry = entries[i];
            stack.Push(entry);
          }
          entry = stack.Pop();// only display the last record
          Console.WriteLine("[Index]\t" + entry.Index +
                        "\n[EventID]\t" + entry.InstanceId +
                        "\n[TimeWritten]\t" + entry.TimeWritten +
                        "\n[MachineName]\t" + entry.MachineName +
                        "\n[Source]\t" + entry.Source +
                        "\n[UserName]\t" + entry.UserName +
                        "\n[Message]\t" + entry.Message +
                        "\n---------------------------------------------------\n");
        }
        else
        {
          Console.WriteLine("Usage:glog.exe system(application,security)\n");
        }
      }
      else
      {
        Console.WriteLine("Usage:glog.exe system(application,security)\n");
      }
      Console.WriteLine("end");
      Console.ReadLine();
    }
  }
}
```
  
- [제어판에서 이벤트 로그 설정 변경하기](http://jacking.tistory.com/1166)
- [이벤트 보관 로그](http://technet.microsoft.com/ko-kr/library/cc749339.aspx)
  