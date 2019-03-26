---
layout: post
title: C# - System.Lazy
published: true
categories: [.NET]
tags: c# net System.Lazy Lazy
---
## System.Lazy<T> ?
- System.Lazy<T> 는 지연 초기화를 지원하는 클래스이다. .NET Framework 4.0 에서 추가
- 사용하는 경우
    - 멀티 스레드 환경에서 공유 리소스의 초기화를 스레드 세이프하게 하고 싶을 때
    - 초기화에 시간이 걸리는(또는 많은 하드웨어 자원을 사용하는) 클래스가 있을 때 이것을 사용할지 안할지 알 수 없는 경우에 사용한다.
  
  
  
## 간단 예제
- http://weblogs.asp.net/gunnarpeipman/archive/2009/05/19/net-framework-4-0-using-system-lazy-lt-t-gt.aspx
  
```
static void Main(string[] args)
{
    var lazyString = new Lazy<string>(
        () =>
        {
            // Here you can do some complex processing
            // and then return a value.
     Console.Write("Inside lazy loader");
            return "Lazy loading!";
        });
    Console.Write("Is value created: ");
    Console.WriteLine(lazyString.IsValueCreated);

    Console.Write("Value: ");
    Console.WriteLine(lazyString.Value);

    Console.Write("Value again: ");
    Console.WriteLine(lazyString.Value);

    Console.Write("Is value created: ");
    Console.WriteLine(lazyString.IsValueCreated);

    Console.WriteLine("Press any key to continue ...");
    Console.ReadLine();
}
```
  
<pre>
Is value created: False
Inside lazy loader
Value: Lazy loading!
Value again: Lazy loading!
Is value created: True
Press any key to continue …
</pre>
  
  
  
## 멀티스레드에서 공유 리소스 초기화 일반 vs Lazy<T>
- double-checked locking
  
```
class Hoge
{
  volatile HeavyObject _heavy;
  readonly object _lock = new object();

  public HeavyObject Heavy
  {
    get
    {
      if (_heavy == null)
      {
        lock (_lock)
        {
          if (_heavy == null)
          {
            _heavy = new HeavyObject();
          }
        }
      }

      return _heavy;
    }
  }
}
```
  
- race-to-initialize
  
```
class Hoge
{
  volatile HeavyObject _heavy;

  public HeavyObject Heavy
  {
    get
    {
      if (_heavy == null)
      {
        var heavy = new HeavyObject();
        Interlocked.CompareExchange(ref _heavy, heavy, null);
      }

      return _heavy;
    }
  }
}
```
  
- Lazy<T> 사용
  
```
var lazy1 = new Lazy<HeavyObject>(() => new HeavyObject(), true); 

//또는

var lazy1 = new Lazy<HeavyObject>(() => new HeavyObject(), LazyThreadSafetyMode.ExecutionAndPublication); 


class Hoge
{
  Lazy<HeavyObject> _heavy = new Lazy<HeavyObject>(() => new HeavyObject(), LazyThreadSafetyMode.ExecutionAndPublication);

  public HeavyObject Heavy
  {
    get
    {
      return _heavy.Value;
    }
  }
}
```
  
