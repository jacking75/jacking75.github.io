---
layout: post
title: C# - async-await에서 lock 사용하기
published: true
categories: [.NET]
tags: .net c# async lock
---
[출처](https://qiita.com/wipiano/items/480fe56dc934be86b7cd  )  
  
C#에서 비동기 메소드에서는 lock을 쓸 수 없다. 이 글은 그래도 lock을 사용 하고 싶을 때를 위한 것이다.  
  
lock이 필요한 경우를 예를 들면 아래처럼 더블 체크 락킹을 하고 싶을 때이다.  
  
```
// /이것이 여러 스레드에서 비동기에게 불리는
private static async ValueTask 조건을만족하면무엇인가하는Async()
{
    if (조건)
    {
        lock (_lockObj) 
        {
            if (조건)
            {
                await 꽤무거운IO작업Async();
            }
        }
    }
}
```
  
lock 스테이트먼트는 비동기 메서드 내에서 사용할 수 없기 때문에, 실제로는 이 코드는 컴파일이 되지 않는다.  
  
위 문제를 간단하게 처리할 수 있는 방법 중 하나로 세마포어를 사용한다.  
보통 세마포어는 프로세스 간 동기화에 사용하지만 .NET에는 프로세스 내에서 이용하기 위한 SemaphoreSlim 클래스가 있다.  
  
SemaphoreSlim 클래스에는 비동기로 기다릴 수 있는 WaitAsync() 메소드가 있다.  
이것을 사용하면 완전히 비동기에서 lock을 사용할 수 있다.  
  
범용적으로 사용할 수 있게 AsyncLock 이라는 이름으로 클래스를 만들어두면 좋을 것이다.  
```
/// async인 맥락에서의 lock을 제공한다.  
/// Lock 해제를 위해 반드시 처리 완료 후에 LockAsync가 생성한 IDisposable을 Dispose 한다. 
public sealed class AsyncLock
{
    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

    public async Task<IDisposable> LockAsync()
    {
        await _semaphore.WaitAsync();
        return new Handler(_semaphore);
    }

    private sealed class Handler : IDisposable
    {
        private readonly SemaphoreSlim _semaphore;
        private bool _disposed = false;

        public Handler(SemaphoreSlim semaphore)
        {
            _semaphore = semaphore;
        }

        public void Dispose()
        {
            if (!_disposed)
            {
                _semaphore.Release();
                _disposed = true;
            }
        }
    }
}
```
  
사용할 때는, lock 구문 대신 using 구문을 사용한다. 이것에 의해서, semaphore 관리를 잊고 lock 이라는 의미를 부여한 코드를 사용한다.  
  
예를 들어, 처음의 예제 코드에서는 아래처럼 사용할 수 있다.  
```
private static readonly s_lock = new AsyncLock();
private static async Task 조건을만족하면무엇인가하는Async()
{
    if (조건)
    {
        using (await s_lock.LockAsync()) 
        {
            if (조건)
            {
                await 꽤무거운IO작업Async();
            }
        }
    }
}
```
  