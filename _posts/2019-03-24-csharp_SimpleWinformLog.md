---
layout: post
title: C# - Simple Winform Logger
published: true
categories: [.NET]
tags: c# .net log
---
## 로그 클래스. DevLog
  
```
//
public enum LOG_LEVEL
{
    TRACE,
    DEBUG,
    INFO,
    WARN,
    ERROR
}

public class DevLog
{
    static System.Collections.Concurrent.ConcurrentQueue<string> logMsgQueue = new System.Collections.Concurrent.ConcurrentQueue<string>();

    static LOG_LEVEL 출력가능_로그레벨 = new LOG_LEVEL();

    static public void Init(LOG_LEVEL logLevel)
    {
        출력가능_로그레벨 = logLevel;
    }

    static public void ChangeLogLevel(LOG_LEVEL logLevel)
    {
        출력가능_로그레벨 = logLevel;
    }

    static public void Write(string msg, LOG_LEVEL logLevel = LOG_LEVEL.TRACE, 
                            [CallerFilePath] string fileName = "",
                            [CallerMemberName] string methodName = "",
                            [CallerLineNumber] int lineNumber = 0)
    {
        if (출력가능_로그레벨 <= logLevel)
        {
            logMsgQueue.Enqueue(string.Format("{0}:{1}| {2}", DateTime.Now, methodName, msg));
        }
    }

    static public bool GetLog(out string msg)
    {
        if (logMsgQueue.TryDequeue(out msg))
        {
            return true;
        }

        return false;
    }

}
```
  
  
## 폼에서 타이머를 이용하여 로그를 가져와서 리스트박스에 출력한다.
  
```
private void OnProcessTimedEvent(object sender, EventArgs e)
{
    // 너무 이 작업만 할 수 없으므로 일정 작업 이상을 하면 일단 패스한다.
    int logWorkCount = 0;

    while (true)
    {
        string msg;

        if (CommonLib.DevLog.GetLog(out msg))
        {
            ++logWorkCount;

            if (listBoxLog.Items.Count > 100)
            {
                listBoxLog.Items.Clear();
            }

            listBoxLog.Items.Add(msg);
            listBoxLog.SelectedIndex = listBoxLog.Items.Count - 1;
        }
        else
        {
            break;
        }

        if (logWorkCount > 7)
        {
            break;
        }
    }
}
```
  
  
  
## 로그 처리용 타이머 정의 
  
```
System.Windows.Threading.DispatcherTimer workProcessTimer = new System.Windows.Threading.DispatcherTimer();
workProcessTimer.Tick += new EventHandler(OnProcessTimedEvent);
workProcessTimer.Interval = new TimeSpan(0, 0, 0, 0, 32);
workProcessTimer.Start();
```
  
  
  
## 로그 사용
  
```
DevLog.Write(string.Format("새로 등록"), LOG_LEVEL.INFO);
```
  
