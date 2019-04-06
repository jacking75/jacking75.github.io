---
layout: post
title: C# - Fluent Scheduler
published: true
categories: [.NET]
tags: c# .net FluentScheduler
---
## 기본
- 주 기능: 지정된 시간에 실행해야 할 Job을 등록해 놓으면 해당 시간이 되면 Job을 실행한다.
- 오픈 소스: https://github.com/jgeurts/FluentScheduler
- 설치: Nuget으로 쉽게 설치할 수 있다.
  
  
  
## 예제 코드
1) 현재 등록된 job 개수
  
```
// 만약 이전에 job을 등록한적이 없다면
// TaskManager.Initialize(new Registry());
// 로 초기화 해 준후 사용해야 한다.
var count = TaskManager.AllSchedules.Count();
```
  
2) 등록된 모드 Job 정보 보기
  
```
var allJobs = TaskManager.AllSchedules;
foreach (var job in allJobs)
{
    var info = string.Format("Name:{0}, 실행예정시간:{1}", job.Name, job.NextRunTime.ToString("yyyy MM-dd HH:mm:ss"));
}
```
  
3) 모든 Job 삭제
  
```
var jobNameList = new List<string>();

var allJobs = TaskManager.AllSchedules;
foreach (var job in allJobs)
{
    jobNameList.Add(job.Name);
}

foreach (var name in jobNameList)
{
    TaskManager.RemoveTask(name);
}
```
  
4) 매번 실행되는 Job
  
```
TaskManager.AddTask(
            () => DevLog.Write("(3초마다) Runner1 " + DateTime.Now), x => x.WithName("Runner1").ToRunEvery(3).Seconds()
            );
```
  
5) 한번만 실행되는 Job
  
```
TaskManager.AddTask(
            () => DevLog.Write("(3초 후에 딱 한번)OnceRunner1 " + DateTime.Now), x => x.WithName("OnceRunner1").ToRunOnceIn(3).Seconds()
            );
```
  
6) 매일 특정 시간에만 실행되는 Job
  
```
TaskManager.AddTask(
            () => DevLog.Write("(매일 0시 0분 0초 마다)1DayJob " + DateTime.Now), x => x.WithName("1DayJob").ToRunEvery(1).Days().At(0, 0)
            );
```
  
7) 등록과 동시 실행 후 매번 실행
  
```
TaskManager.AddTask(
            () => DevLog.Write("(5초 마다)Runner2 " + DateTime.Now), x => x.WithName("Runner2").ToRunNow().AndEvery(5).Seconds()
            );
```
  
8) 매주 특정 요일에만 실행
  
```
// 지금으로 부터 1주일 후 목요일
TaskManager.AddTask(
            () => DevLog.Write("(매주 목요일마다)Runner3 " + DateTime.Now), x => x.WithName("Runner3").ToRunEvery(1).Weeks().On(DayOfWeek.Thursday)
            );

// 이번 주 목요일
TaskManager.AddTask(
            () => DevLog.Write("(매주 목요일마다)Runner3 " + DateTime.Now), x => x.WithName("Runner3").ToRunEvery(0).Weeks().On(DayOfWeek.Thursday)
            );
```
  
9) 한번에 아주 많은 수의 job을 등록할 때  
TaskManager.AddTask를 사용하여 많은 수(1000만개 이상)의 job을 등록할 때 등록이 끝나기 전에 job이 실행되어 버리면 추가하는 도중 예외가 발생한다. 이유는 AddTask 함수는 새로운 job이 등록될 때마다 정렬을 하고 있어서 그렇다. 그래서 한번에 많은 job을 등록할 때는 Registry 클래스를 사용하여 job을 등록해야 한다.  
```
var registry = new MyRegistry();
registry.Init(jobCount);

TaskManager.Initialize(registry);


public class MyRegistry : Registry
{
    public void Init(int jobCount)
    {
        var rand = new Random();

        for (int i = 0; i < jobCount; ++i)
        {
            int delayTimeSec = rand.Next(10, 30);

            Schedule( () => newTask.Execute()).WithName("dummyTask_" + i).ToRunOnceIn(delayTimeSec).Seconds();
        }
    }
}

public class MyTask : ITask
{
    public void Execute()
    {
    }
}
```
  