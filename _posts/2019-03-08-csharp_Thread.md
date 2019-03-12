---
layout: post
title: C# - Thread
published: true
categories: [.NET]
tags: c# .net Thread
---
## 현재 스레드의 ID 얻기
  
```
System.Threading.Thread.CurrentThread.ManagedThreadId
```
  
  
  
## 스레풀의 최소, 최대 스레드 개수 알기
  
```
int workThreads = 0;
int iocpThreads = 0;

// 최소 스레드 개수
System.Threading.ThreadPool.GetMaxThreads(out workThreads, out iocpThreads);

// 최대 스레드 개수
System.Threading.ThreadPool.GetMinThreads(out workThreads, out iocpThreads);
```
  
  
  
## 스레드 사용하기
  
```
System.Threading.Thread TimeThread = null;

// 생성, 시작
IsThread1Running = true;

TimeThread = new System.Threading.Thread(this.TimeThreadProcess);
TimeThread.Start();

// 종료
IsThread1Running = false;

if (TimeThread.IsAlive)
{
   TimeThread.Join();
}

// 스레드용 함수
void TimeThreadProcess()
{
   while (IsThread1Running)
   {
      System.Threading.Thread.Sleep(100);

      if (IsRunTest == false)
      {
         continue;
      }

      var Diff = EndTime.Subtract(DateTime.Now);

      if (Diff.Seconds <= 0)
      {
         IsRunTest = false;
      }
   }
}
```
  
  
  
## 스레드 사용하기. 스레드 함수에 파라미터 전달
- 파라미터 전달은 1개만 가능.
  
```
Thread myThread = new Thread( new ParameterizedThreadStart(threadFunc) );
myThread.Start(파라미터(매개변수));

private void threadFunc(object o)
{
    ...
}
```
  
  
  
## 스레드풀 사용하기
  
```
class Program
{
    static void Main(string[] args)
    {
        System.Threading.ThreadPool.QueueUserWorkItem(new System.Threading.WaitCallback(키보드입력조사), null);

        while (true)
        {
            ......
            System.Threading.Thread.Sleep(128);
        }
    }

    static void 키보드입력조사(object userState)
    {
        while (true)
        {

        }
    }
}
```
  
  
  
## SpinLock
- https://msdn.microsoft.com/ja-jp/library/system.threading.spinlock(v=vs.110).aspx
  
  
  
## ReaderWriterLock
- ReaderWriterLock은 여러 스레드에서 공유 자원에 접근할 때 동기화하기 위해 사용한다.
- Monitor를 사용한 동기와 달리 ReaderWriterLock은 읽기 접근에 대해서는 복수의 쓰레드에 허가하고 쓰기 접근에 대해서는 하나의 스레드만 허가한다.
- 스레드가 읽기 위한 lock(리더 락)을 취득할 때는 ReaderWriterLock.AcquireReaderLock 메서드를, 쓰기 접근을 할 때는 ReaderWriterLock.AcquireWriterLock 메서드를 호출한다.
- 예
  
```
using System.Threading;

private static ReaderWriterLock rwl = new ReaderWriterLock();

// 복수의 스레드에서 접근할 공유 리소스
private static string resource = "0123456789";

public static void Main()
{
    for (int i = 0; i < 200; i++)
    {
        if (i % 2 == 0)
        {
            (new Thread(new ThreadStart(ReadFromResource))).Start();
        }
        else
        {
            (new Thread(new ThreadStart(WriteToResource))).Start();
        }
    }

    Console.ReadLine();
}

// 공유 리소스를 읽는다
private static void ReadFromResource()
{
    rwl.AcquireReaderLock(Timeout.Infinite);

    Console.WriteLine(resource);

    rwl.ReleaseReaderLock();
}

// 공유 리소스를 쓴다
private static void WriteToResource()
{
    rwl.AcquireWriterLock(Timeout.Infinite);

    string s = resource.Substring(0, 1);
    resource = resource.Substring(1);
    Thread.Sleep(1);
    resource += s;

    rwl.ReleaseWriterLock();
}
```
  
- 리더 락을 취득 중에 공유 자원에 입력할 필요가 생겼을 때 ReaderWriterLock.UpgradeToWriterLock 메서드를 사용하여 라이터 락으로 업그레이드할 수 있다.(ReaderWriterLock.
UpgradeToWriterLock 메서드를 호출하면 ReaderWriterLock.DowngradeFromWriterLock 메소드로 리더 락을 복원한다.)
- ReaderWriterLock.ReleaseLock 메서드도 락을 해방하지만 이때는 스레드가 락을 취득한 횟수에 상관 없이 곧바로 락을 해방한다.
- ReleaseLock 메서드가 반환한 LockCookie 객체를 RestoreLock 메서드에 사용하여 해방한 락을 복원하는 것도 할 수 있다. 이 것을 이용하면 락을 일시적으로 해방할 수 있다.
- 또 WriterSeqNum 속성에서 현재의 writer 시퀀스 번호를 기억해두고, 이것을 AnyWritersSince 메서드에 사용하여 writer 락을 취득한 스레드가 있는지 알아볼 수 있다.
- 만약 Writer 락을 취득한 스레드가 있으면 공유 자원의 내용이 변경됐다고 판단할 수 있다. 이를 이용하면 예컨대 리더 락 해방 전에 공유 자원의 내용을 로컬 변수 등에 보관하고, 그 후 다시 Writer 락을 취득한 이후 공유 자원의 내용이 변경되어 있으면 공유 리소스를 읽고, 변경되지 않으면 보간하고 있는 내용을 사용할 수 있다.
- 아래는 UpgradeToWriterLock, DowngradeFromWriterLock 메서드, ReleaseLock 메서드, RestoreLock 메서드, WriterSeqNum 프로 퍼티, AnyWritersSince 메서드의 구체적인 사용 예를 나타낸다.
    - UpAndDownGrade 메서드에서 UpgradeToWriterLock, DowngradeFromWriterLock 메서드를 ReleaseAndRestore 메서드로 ReleaseLock 메서드, RestoreLock 메서드, WriterSeqNum프로 퍼티, AnyWritersSince 메서드를 사용하고 있다.
  	
```
using System.Threading;

private static ReaderWriterLock rwl = new ReaderWriterLock();
private static string resource = "0123456789";

public static void Main()
{
    for (int i = 0; i < 200; i++)
    {
        if (i % 2 == 0)
        {
            (new Thread(new ThreadStart(UpAndDownGrade))).Start();
        }
        else
        {
            (new Thread(new ThreadStart(ReleaseAndRestore))).Start();
        }
    }

    Console.ReadLine();
}

// 공유 리소스를 읽고/쓴다
private static void UpAndDownGrade()
{
    rwl.AcquireReaderLock(Timeout.Infinite);

    Console.WriteLine(resource);

    // writer 락으로 업그레이드
    LockCookie lc = rwl.UpgradeToWriterLock(Timeout.Infinite);

    string s = resource.Substring(0, 1);
    resource = resource.Substring(1);
    Thread.Sleep(1);
    resource += s;

    // 리더 락으로 돌아간다
    rwl.DowngradeFromWriterLock(ref lc);

    rwl.ReleaseReaderLock();
}

// 락을 해제하고 복원한다
private static void ReleaseAndRestore()
{
    rwl.AcquireReaderLock(Timeout.Infinite);

    string resourceValue = resource;

    // 현재 writer 시퀸스 번호를 기억해둔다
    int seqNum = rwl.WriterSeqNum;

    // 락을 해제한다. 뒤에서 복원하기 위해 LockCookie를 기억해 둔다.
    LockCookie lc = rwl.ReleaseLock();

    // 락이 해제 되는 것는 잠깐 기다린다
    Thread.Sleep(10);

    // 락을 복원한다
    rwl.RestoreLock(ref lc);

    // 락 해제 중에 다른 스레드가 writer 락을 얻었는가를 조사하여
    // true 이라면 공유 리소스가 변경 되었다고 볼 수 있다
    if (rwl.AnyWritersSince(seqNum))
    {
        Console.WriteLine("({0})", resourceValue);
        resourceValue = resource;
    }

    Console.WriteLine(resourceValue);

    rwl.ReleaseReaderLock();
}
```
  