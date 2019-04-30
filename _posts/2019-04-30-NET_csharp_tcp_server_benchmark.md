---
layout: post
title: C#에서 async-await를 사용하여 TCP Server 만들기
published: true
categories: [.NET]
tags: c# .net asyncawait tcp socket
---
이 글은 [여기](http://qiita.com/tadokoro/items/76085061c38de3d021c0 ) 글을 번역,정리 한 것이다.  
예제 코드는 [여기](https://github.com/hythof/csharp_tcp_server_benchmark )  
  
  
## C# 네트워크 프로그래밍에서 비동기 IO를 사용할 때는 3가지 방법이 있다.  
각 방법 별 성능은 아래와 같다.  
  
| 구현 방법       | 행수 | 난이도 | 초당 요청 수 |
|----------------|------|--------|------------------|
| async/await    | 101  | 쉬움 | 56,916           |
| 비동기 소켓 | 170  | 어려움 | 68,272           |
| ThreadPool     | 98   | 어려움 | 51,129           |
  
<br/>
  
**속도가 중요하면 비동기 소켓**, **쉬운 개발이 중요하면 async 수식자** 로 구분하는 것을 뒷 받침 해주는 결과이다.  
  
  
  
## async 수식자를 사용하면 비동기 처리가 쉽다
다음 코드는 1초간 처리를 지연시키는 Task.Delay(1000)을 합계 100번 실행하고 있지만 처리는 1초 미만으로 끝나고 계산 결과도 올바르게 표시된다.  
기존에서도 ThreadPool.QueueUserWorkItem 등을 사용하면 시간이 많이 걸리는 처리를 백그라운드에서 돌릴 수 있지만 이 예처럼 100개의 비동기 처리를 "다 끝날 때까지 기다리는" 것은 이렇게 쉽게 쓸 수 없다.  
  
ThreadPool은 ThreadPool.SetMinThreads()에서 worker스레드를 동시 접속 수와 동등한 정도로 설정하지 않으면 상기의 성능이 나오지 않았다.또 검증에 사용한 코드를 github에 두고 있다.  
  
```
using System;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    public static void Main()
    {
        var tasks = Enumerable.Range(1, 100).Select(run).ToArray();
        Task.WaitAll(tasks);
        Console.WriteLine(tasks.Sum(x => x.Result));
    }

    static async Task<int> run(int n)
    {
        await timeConsumingWork().ConfigureAwait(false);
        return n * 2;
    }

    static Task timeConsumingWork()
    {
        return Task.Delay(1000);
    }
}
```
  
  
  
## 비동기 소켓을 기존 보다 다루기 쉬워졌다
비동기 소켓은 170줄로 구현하지만 async/await는 101 정도로 간결하게 끝난다.  
또 try/catch의 범위도 비동기 소켓은 콜백 함수를 사용하고 있어서 일괄적으로 예외를 파악해야 하는 반면, async/await쪽은 기존처럼 기술하면 된다.  
  
  
  
## async/await 추천
직감적으로 쓰고 읽을 수 있으므로 C#의 비동기 통신은 async 수식자를 사용하는 것이 좋다. 성능도 나쁘진 않다.  
  