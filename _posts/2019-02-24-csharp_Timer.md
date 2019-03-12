---
layout: post
title: C# - 닷넷프레임워크의 타이머 관련 클래스
published: true
categories: [.NET]
tags: c# .net DispatcherTimer
---
인터넷에 있는 글에서 가져온 것입니다.  
  
## DispatcherTimer
일반 타이머는 메인 스레드가 아닌 다른 스레드를 만들어서 처리 되지만 이것은 실행은 메인 스레드에서 처리 되어 메인 스레드와 동기화가 보장 된다.  
  
#### MSDN의 설명
- 각 Dispatcher 루프의 맨 위에서 DispatcherTimer가 다시 평가된다.  
- 시간 간격이 발생할 때 정확하게 타이머가 실행될지 보장할 수 없지만 시간 간격이 발생하기 전에 타이머가 실행되지 않는 것은 보장할 수 있다.   
- 이것은 DispatcherTimer 작업이 다른 작업처럼 Dispatcher 큐에 있기 때문이다.    
- DispatcherTimer 작업 실행 시기는 큐의 다른 작업 및 해당 우선 순위에 따라 다르다.  
- System.Timers..::.Timer가 WPF 응용 프로그램에서 사용되는 경우 System.Timers..::.Timer는 UI(사용자 인터페이스) 스레드가 아닌 다른 스레드에서 실행된다. 
- UI(사용자 인터페이스) 스레드의 개체에 액세스하려면 Invoke나 BeginInvoke를 사용하여 UI(사용자 인터페이스) 스레드의 Dispatcher로 작업을 게시해야 한다. 
- System.Timers..::.Timer를 사용하는 예제를 보려면 시스템 타이머를 통한 명령 소스 비활성화 샘플을 참조한다. 
- System.Timers..::.Timer 대신 DispatcherTimer를 사용하는 이유는 DispatcherTimer는 Dispatcher와 같은 스레드에서 실행되며 DispatcherTimer에서 DispatcherPriority를 설정할 수 있기 때문이다.
- 개체의 메서드가 타이머에 바인딩될 때마다 DispatcherTimer는 개체를 유지한다.
- 사용 방법  
    - 참조에서 WindowsBase를 추가해야 된다.
  
```
private void dispatcherTimer_Tick(object sender, EventArgs e)
{
	// Updating the Label which displays the current second
	lblSeconds.Content = DateTime.Now.Second;

	// Forcing the CommandManager to raise the RequerySuggested event
	CommandManager.InvalidateRequerySuggested();
}
  
dispatcherTimer = new System.Windows.Threading.DispatcherTimer();
dispatcherTimer.Tick += new EventHandler(dispatcherTimer_Tick);
// 1분. 만약 1초로 하고 싶으면 new TimeSpan(0,0,0,1)
dispatcherTimer.Interval = new TimeSpan(0,0,1);
dispatcherTimer.Start();
```
  
  
## Threading 타이머 사용
동기화가 복잡하지 않고 단일 쓰레드의 경우는 쓰레드 보다 타이머를 사용하는 것이 훨씬 간편하다.  
  
```
....................
void TotalPatch_Callback(Object state)
{
	 ..................
}
...............
// 타이머 생성과 동시에 작동하며 딱 한번만 타이머가 작동한다.
System.Threading.Timer t = new System.Threading.Timer(new System.Threading.TimerCallback(TotalPatch_Callback));
t.Change(0, System.Threading.Timeout.Infinite);
```
  
  
## Timers의 타이머 사용
  
```
Timer aTimer = new System.Timers.Timer(10000); // Hook up the event handler for the Elapsed event.
aTimer.Elapsed += new ElapsedEventHandler(OnTimedEvent); // Only raise the event the first time Interval elapses. aTimer.Enabled = true;

private static void OnTimedEvent(object source, ElapsedEventArgs e)
{
	Console.WriteLine("Hello World!");
}
```
  

