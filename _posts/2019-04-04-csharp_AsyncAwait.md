---
layout: post
title: C# - Async/Await
published: true
categories: [.NET]
tags: c# .net async await thread concurrency
---
- http://csharpstudy.com/CSharp/CSharp-async-await.aspx
- C# 5.0 이 나온 이후 각 써드파티 라이브러리에서 비동기 메소드 지원
  
  
  
## 간편한 비동기 프로그래밍:async/await 
- http://www.simpleisbest.net/post/2013/02/06/About_Async_Await_Keyword_Part_1.aspx
- http://www.simpleisbest.net/post/2013/02/12/About_Async_Await_Keyword_Part_2.aspx
- http://www.simpleisbest.net/post/2013/02/16/About_Async_Await_Keyword_Part_3.aspx
- http://www.simpleisbest.net/post/2013/02/28/About_Async_Await_Keyword_Part_4.aspx 
- http://www.simpleisbest.net/post/2013/03/12/About_Async_Await_Keyword_Part_5.aspx
  
  
  
###async를 사용한 가상 함수 정의
  
```
virtual public async Task<bool> Process()
{
	var task = new TaskCompletionSource<bool>();
	await task.Task;
	return task.Task.Result;
}
```
  
  
  
## Async는 비동기로 동작하지 않는다
- 아래의 코드에서 출력될 문자의 순서는 어떻게 될까?
  
```
async Task WorkThenWait() {
  Thread.Sleep(1000);
  Console.WriteLine("work");
  await Task.Delay(1000);
}

void Demo() {
  var child = WorkThenWait();
  Console.WriteLine("started");
  child.Wait();
  Console.WriteLine("completed");
}
```
  
- 얼핏 생각하기에는  "started", "work", "completed" 순서를 예상하겠지만
    - 결과는 "work", "started", "completed" 이다.
- WorkThenWait()안의 코드가 실행된 스레드는 WorkThenWait()를 호출한 스레드와 같은 스레드이다.
- C#에서는 async 메소드에 있는 코드의 최초 부분은 (호출측의 스레드 상)동기적으로 실행된다.
- 위의 문제를 해결하려면 WorkThenWait()의 첫 코드에 await Task.Yield() 를 추가한다.
    - Task.Yield  http://mvcp.tistory.com/entry/C-SnippetTaskbased-Asynchronous-Pattern-TaskYield
  
  
  
## 결과를 무시한다
  
```
async Task Handler() {
  Console.WriteLine("Before");
  Task.Delay(1000);
  Console.WriteLine("After");
}
```
  
- 위의 코드를 보고 "Before" 출력 후 1초 후에 "After"가 출력된다고 생각하지만....틀렸다.
    - "Before" 후 바로 "After"가 출력된다.
- 이유는 Task.Delay 때문
    - 위 코드의 문제를 해결하기 위해서는 await를 사용해야 한다.
- Task.Delay 예제. 매 초마다 출력: Task.Delay를 사용하지 않은 경우
  
```
private async void button1_Click(object sender, EventArgs e)
{
	for (int i = 0; i < 10; i++)
	{
		await Task.Run(() => System.Threading.Thread.Sleep(1000));
		label1.Text = i.ToString();
	}
}
```
  
- Task.Delay 예제. 매 초마다 출력: Task.Delay 사용
  
```
private async void button1_Click(object sender, EventArgs e)
{
	for (int i = 0; i < 10; i++)
	{
		await Task.Delay(1000);
		label1.Text = i.ToString();
	}
}
```
  
  
  
## Async void 메소드
- 뒷 사람을 위해서 async void 사용을 자제하는 것이 좋다
  
```
async void ThrowExceptionAsync() {
  throw new InvalidOperationException();
}

public void CallThrowExceptionAsync() {
  try {
	ThrowExceptionAsync();
  } catch (Exception) {
	Console.WriteLine("Failed");
  }
}
```
  
- 결과가 반환되지 않으므로 위의 코드에서  catch 부분이 실행되지 않음!!!
  
  
  
## Async void 람다 함수
  
```
Parallel.For(0, 10, async i => {
  await Task.Delay(1000);
});
```
  
- 이 코드는 1초 후에 완료할까? 아니면 바로 완료될까?
- 람다 함수가 For의 Action 델리게이트가 되어 async void로 된다.
  
  
  
## nasted된 task
- 아래의 코드는 2개의 출력 사이에 1초를 기다릴까?
  
```
Console.WriteLine("Before");
await Task.Factory.StartNew(
  async () => { await Task.Delay(1000); });
Console.WriteLine("After");
```
  
- 기다리지 않는다.
- StartNew는 델리게이트를 받고, Task<T>를 반환한다. T는 델리게이트에 의해 반환되는 타입이다.
    - 위의 코드에서는 Task<Task>를 반환한다.
- await의 사용은 (바로 내측의 task를 반환하는)외측의 task 완료를 기다리고, 그 후는 내측의 task는 무시한다.
- StartNew 대신에 Task.Run을 사용하면(혹은 람다 함수 내에서 async와 await를 Drop Shipping 하던가) 문제를 해결할 수 있다.
  
  
  
## 비동기로 실행 되지 않는다
  
```
async Task FooAsync() {
  await Task.Delay(1000);
}
void Handler() {
  FooAsync().Wait();
}
```
  
- 위의 코드는 태스트를 만든 후 Wait를 사용하여 그것이 완료될 때까지 블럭
    - 비동기로 실행하고 싶으면 FooAsync().Wait(); 에서 Wait()를 제거하면 된다.
  
  
  
## ASP.NET에서 SynchronizationContext
- 아래 코드는 문제가 없을까?
  
```
public class HomeController : Controller
{
	async Task DoAsync()
	{
		await Task.Delay(TimeSpan.FromSeconds(3));
	}

	public ActionResult Index()
	{
		DoAsync();
		return View();
	}
}
```
  
- 위의 코드는 처음에는 문제가 없지만 좀 더 시간이 지나면 UnobservedTaskException 예외가 발생한다.
- 문제는 await Task.Delay에 있다.
- await에 의해 POST 하는 곳의 Context가 사라지기 때문이다.
    - 비동기 처리를 기다리지 않으므로 View 표시까지 모두 완료해서 Context가 소멸한다.
    - 이 후에 DoAsync 내의 await가 완료하여 계속해서 POST를 시작하지만 이 때는 Context가 사라져서 없다.
- 해결책은 다음과 같다 
  
```
public class HomeController : Controller
{
	async Task DoAsync()
	{
		Trace.WriteLine("start"); 

		await Task.Delay(TimeSpan.FromSeconds(3)); 

		Trace.WriteLine("end"); 
	}

	public async Task<ActionResult> Index()
	{
		GC.Collect();

		await DoAsync(); 
		return View();
	}
}
```
  
혹은
  
```
public class HomeController : Controller
{
	async Task DoAsync()
	{
		Trace.WriteLine("start"); 

		await Task.Delay(TimeSpan.FromSeconds(3)).ConfigureAwait(false); 

		Trace.WriteLine("end"); 
	}

	public ActionResult Index()
	{
		GC.Collect();

		var _ = DoAsync(); 
		return View();
	}
}
```
  
  
  
## 참고
- [(일어)C#과 F#의 Async: C#의 비동기 함정](https://gist.github.com/pocketberserker/5565303)
  
