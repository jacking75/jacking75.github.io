---
layout: post
title: C# - Thread Local
published: true
categories: [.NET]
tags: c# net ThreadLocal thread
---
- 닷넷 프레임워크 4.0 에서 추가
    - 이전에는 비슷한 것으로 ThreadStatic 속성이 있었음
        - 그러나 static 멤버에만 사용 가능.
        - 멤버의 값은 보통 그 타입의 기본 값으로 초기화된다. 초기 값을 설정해도 무시된다.
- ThreadLocal 클래스는 위의 문제를 모두 해결  
- 사용 방법
  
```
ThreadLocal<int> _id = new ThreadLocal<int>(() => Thread.CurrentThread.ManagedThreadId);
```
  
- () => Thread.CurrentThread.ManagedThreadId 를 ValueFactory 라고 부른다. 각 스레드에 처음 접근될 때 평가 된다.
- 예제 코드
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

class ThreadLocalDemo
{

        // Demonstrates:
        //      ThreadLocal(T) constructor
        //      ThreadLocal(T).Value
        //      One usage of ThreadLocal(T)
        static void Main()
        {
            // Thread-Local variable that yields a name for a thread
            ThreadLocal<string> ThreadName = new ThreadLocal<string>(() =>
            {
                return "Thread" + Thread.CurrentThread.ManagedThreadId;
            });

            // Action that prints out ThreadName for the current thread
            Action action = () =>
            {
                // If ThreadName.IsValueCreated is true, it means that we are not the
                // first action to run on this thread.
                bool repeat = ThreadName.IsValueCreated;

                Console.WriteLine("ThreadName = {0} {1}", ThreadName.Value, repeat ? "(repeat)" : "");
            };

            // Launch eight of them.  On 4 cores or less, you should see some repeat ThreadNames
            Parallel.Invoke(action, action, action, action, action, action, action, action);

            // Dispose when you are done
            ThreadName.Dispose();
        }
}
```
  
- 설명: http://mvcp.tistory.com/167
- ThreadLocal과 random
    - random은 스레드 세이프 하지 않으므로 기본적으로 락을 걸어야 한다.
    - 그러나 ThreadLocal과 같이 사용하면 문제 없음
    - 출처: http://csharpindepth.com/Articles/Chapter12/Random.aspx
  
```
public static class RandomProvider
{    
    private static int seed = Environment.TickCount;

    private static ThreadLocal<Random> randomWrapper = new ThreadLocal<Random>(() =>
        new Random(Interlocked.Increment(ref seed))
    );

    public static Random GetThreadRandom()
    {
        return randomWrapper.Value;
    }
}
```
  
- MSDN 문서
    - (기계 번역)http://msdn.microsoft.com/ko-kr/library/dd642243.aspx
    - (일본어) http://msdn.microsoft.com/ja-jp/library/vstudio/dd642243.aspx
  