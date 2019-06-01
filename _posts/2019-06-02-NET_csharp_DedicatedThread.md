---
layout: post
title: C# - 게임 서버에서 프레임 단위로 클라이언트 요청을 처리한다. async-await 사용
published: true
categories: [.NET]
tags: c# .net asyncawait
---
[출처](http://engineering.grani.jp/entry/2017/06/02/190012)    
    
```
/*
var thread = GameLoopThreadPool.GetThread();
while (true)
{
    // frameAction 안에서 AI의 행동 선택 처리나 쌓인 커맨드를 기초로 데미지를 주거나 회복 시키는 코드가 동작
    var shouldContinue = frameAction(this);
    if (!shouldContinue) break;
    await thread.NextFrame();
}
*/

public class GameLoopThread
{
    const int InitialSize = 16;

    Thread thread; // Thread를 사용한다.
    int threadNumber = 0;

    readonly object gate = new object();
    int frameMilliseconds;

    // 2개의 배열(축소하지 않는)을 큐로 사용하여 lock을 최소한으로 하고 배열 확장/축소 비용, 그리고 루프 속도를 번다
    bool isQueueA = true;
    int queueACount = 0;
    Action[] queueA = new Action[InitialSize];
    int queueBCount = 0;
    Action[] queueB = new Action[InitialSize];

    public GameLoopThread(int frameMilliseconds, int threadNumber)
    {
        this.frameMilliseconds = frameMilliseconds;
        this.thread = new Thread(new ParameterizedThreadStart(RunInThread), 32 * 1024) // maxStackSize은 작아도 상관 없다
        {
            Name = "GameLoopThread" + threadNumber,
            Priority = ThreadPriority.Normal,
            IsBackground = true
        };
        this.thread.Start(this);
    }

    static void EnsureCapacity(ref Action[] array, int count)
    {
        if (count == array.Length)
        {
            var newLength = count * 2;
            var newArray = new Action[newLength];
            Array.Copy(array, newArray, array.Length);
            array = newArray;
        }
    }

    static void RunInThread(object objectSelf)
    {
        var self = (GameLoopThread)objectSelf;
        self.Run();
    }

    void Run()
    {
        var sw = Stopwatch.StartNew();
        while (true) // 실제 코드에서는 밖에서 종료 할 있도록 문을 준비하고 있다
        {
            sw.Restart();
            Action[] useQueue;
            int useQueueCount;

            lock (gate)
            {
                if (isQueueA)
                {
                    useQueue = queueA;
                    useQueueCount = queueACount;
                    queueACount = 0;
                    isQueueA = false;
                }
                else
                {
                    useQueue = queueB;
                    useQueueCount = queueBCount;
                    queueBCount = 0;
                    isQueueA = true;
                }
            }

            for (int i = 0; i < useQueueCount; i++)
            {
                useQueue[i].Invoke(); // 이게 예외를 내는 것은 허락하지 않는다. 라는 것을 호출 측에서 보증해 준다
                useQueue[i] = null;
            }
            useQueue = null;

            sw.Stop();

            if (useQueueCount != 0)
            {
                // Datadog 에 처리 시간을 모니터링을 넣어 둔다
                var tags = new[] { "gameloopthreadnumber:no-" + threadNumber };
                DatadogStats.Default.Gauge("GameLoopThreadRun.avg", sw.Elapsed.TotalMilliseconds, sampleRate: 0.1, tags: tags);
                DatadogStats.Default.Increment("GameLoopThreadRun.count", sampleRate: 0.1, tags: tags);
                DatadogStats.Default.Gauge("GameLoopThreadRun.ProcessCount", useQueueCount, sampleRate: 0.1, tags: tags);
            }

            // 정확한 게임 루프를 따르는 경우는  시간의 차리를 취하면서 Sleep(1) 으로 처리 회전 여부를 넣을지 어떨지 체크를 넣는다
            // 루프를 돌리는 간격의 애매함을 허럭하는 경우는 이대로도 문제 없다
            Thread.Sleep(frameMilliseconds);
        }
    }

    void RegisterCompletion(Action continuation)
    {
        lock (gate)
        {
            if (isQueueA)
            {
                EnsureCapacity(ref queueA, queueACount);
                queueA[queueACount++] = continuation;
            }
            else
            {
                EnsureCapacity(ref queueB, queueBCount);
                queueB[queueBCount++] = continuation;
            }
        }
    }

    public GameLoopAwaiter NextFrame()
    {
        return new GameLoopAwaiter(this);
    }

    public struct GameLoopAwaiter : ICriticalNotifyCompletion, INotifyCompletion
    {
        GameLoopThread thread;

        public GameLoopAwaiter(GameLoopThread thread)
        {
            this.thread = thread;
        }

        public bool IsCompleted => false;

        public void GetResult()
        {
        }

        public GameLoopAwaiter GetAwaiter()
        {
            return this;
        }
		
        // await를 호출하면 OnCompleted 시리즈가 호출되어 GameLoopThread 에 Action을 읽는다
        public void OnCompleted(Action continuation)
        {
            thread.RegisterCompletion(continuation);
        }

        public void UnsafeOnCompleted(Action continuation)
        {
            thread.RegisterCompletion(continuation);
        }
    }
}
```
  