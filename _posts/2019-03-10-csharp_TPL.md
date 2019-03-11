---
layout: post
title: C# - TPL
published: true
categories: [.NET]
tags: c# net tpl task thread
---
닷넷 프레임워크 4.0에서 추가된 병렬 프로그래밍 라이브러리  
[여기의 글](http://xin9le.net/tpl-intro )을 정리  
  
  
## Parallel.ForEach/For
- Parallel.ForEach
  
```
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

namespace Sample01_ParallelForEach
{
	class Program
	{
		static void Main()
		{
			//----- 데이터 생성
			var collection = new int[]{ 10, 20, 30, 40, 50 };

			//----- 일반적인 루프
			var stopwatch = Stopwatch.StartNew();
			foreach (var item in collection)
			{
				Console.WriteLine(item);
				Thread.Sleep(1000);
			}
			stopwatch.Stop();
			Console.WriteLine(stopwatch.ElapsedMilliseconds.ToString("Normal : 0[ms]"));

			//----- 병렬 루프
			stopwatch.Restart();
			Parallel.ForEach(collection, item =>
			{
				Console.WriteLine(item);
				Thread.Sleep(1000);
			});
			stopwatch.Stop();
			Console.WriteLine(stopwatch.ElapsedMilliseconds.ToString("Parallel : 0[ms]"));
		}
	}
}
```
  
<pre>
//----- 결과 : 환경에 따라서 다소 결과가 다른 경우도 있다.
10
20
30
40
50
Normal : 5002[ms]
10
20
40
30
50
Parallel : 1010[ms]
</pre>
  
- Parallel.For
  
```
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

namespace Sample02_ParallelFor
{
	class Program
	{
		static void Main()
		{
			//----- 데이터 생성
			var collection = new int[]{ 10, 20, 30, 40, 50 };

			//----- 일반적인 루프
			var stopwatch = Stopwatch.StartNew();
			for (int index = 0; index < collection.Length; index++)
			{
				Console.WriteLine(collection[index]);
				Thread.Sleep(1000);
			}
			stopwatch.Stop();
			Console.WriteLine(stopwatch.ElapsedMilliseconds.ToString("Normal : 0[ms]"));

			//----- 병렬 루프
			stopwatch.Restart();
			Parallel.For(0, collection.Length, index =>
			{
				Console.WriteLine(collection[index]);
				Thread.Sleep(1000);
			});
			stopwatch.Stop();
			Console.WriteLine(stopwatch.ElapsedMilliseconds.ToString("Parallel : 0[ms]"));
		}
	}
}
```
  
<pre>
10
20
30
40
50
Normal : 5002[ms]
10
30
50
20
40
</pre>
  
  
  
## Parallel 루프 중단
- Paralle.For/ForEach 는 기본적으로 루프 중에 break 문으로 중단할 수 없다.
- ParallelLoopState.Stop
   - Stop 이 호출될 때 실행 중인 작업이 중단된다.
   - 다른 스레드에서 Stop 호출을 확인하기 위해서 ParallelLoopState.IsStopped 를 사용한다.
  
```
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

namespace Sample04_StopLoop
{
	class Program
	{
		static void Main()
		{
			var stack = new ConcurrentStack<long>();
			Parallel.For(0, 100000, (index, state) =>
			{
				if (index < 50000) stack.Push(index);
				else               state.Stop();
			});
			Console.WriteLine("개수: " + stack.Count);
		}
	}
}
```
  
- ParallelLoopState.Break
   - 작업 중 Break가 호출되면 현재 작업 중인 것은 처리하고 다음부터 중단된다.
  
```
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

namespace Sample05_BreakLoop
{
	class Program
	{
		static void Main()
		{
			var stack = new ConcurrentStack<long>();
			Parallel.For(0, 100000, (index, state) =>
			{
				stack.Push(index);
				if (index >= 50000)
				{
					state.Break();
					Console.WriteLine("Index : " + index);
					Console.WriteLine("LowestBreakIteration : " + state.LowestBreakIteration);
				}
			});
			Console.WriteLine("개수 : " + stack.Count);
		}
	}
}
```
  
<pre>
결과
ndex : 50000
LowestBreakIteration : 50000
Index : 50016
LowestBreakIteration : 50000
개수 : 50002
</pre>
  
- ParallelLoopResult
   - 병렬 루프가 작업 도중 정지된 것인지 알 수 있다.
  
```
using System.Threading.Tasks;

namespace Sample06_LoopResult
{
	class Program
	{
		static void Main()
		{
			var result = Parallel.For(0, 100000, (index, state) =>
			{
				//----- Do Something...
			});

			if (result.IsCompleted)
			{
				//----- 모든 루프가 실행 되었다
			}
			else if (result.LowestBreakIteration.HasValue)
			{
				//----- Break 되었다
			}
			else
			{
				//----- Stop 되었다
			}
		}
	}
}
```
  
  
  
## 스레드 로컬 변수
  
```
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace Sample07_ThreadLocalVariables
{
	class Program
	{
		static void Main()
		{
			int total = 0;
			Parallel.ForEach
			(
				Enumerable.Range(1, 100), // 병렬 처리를 할 소스
				() => 0,                  //스레드 로컬 변수 초기화 값
				(value, state, local) => local += value, // 루프 작업
				local => Interlocked.Add(ref total, local) // 루프 작업 완료 후 처리
			);
			Console.WriteLine(total);
		}
	}
}
```
  
  
  
## 예외 처리
  
```
using System;
using System.Linq;
using System.Threading.Tasks;

namespace Sample08_LoopException
{
	class Program
	{
		static void Main()
		{
			try
			{
				Parallel.ForEach(Enumerable.Range(0, 100), value =>
				{
					if (value % 2== 0)  throw new InvalidOperationException();
					else                throw new ArgumentException();
				});
			}
			catch (AggregateException ex)
			{
				foreach (var exception in ex.InnerExceptions)
					Console.WriteLine(exception.GetType().Name);
			}
		}
	}
}
```
  
<pre>
결과
InvalidOperationException
InvalidOperationException
ArgumentException
ArgumentException
InvalidOperationException
InvalidOperationException
ArgumentException
ArgumentException
InvalidOperationException
</pre>
  
-내부에서 발생한 예외는 AggregateException의 InnerExceptions 프로퍼티에 모두 저장된다. 이후 catch 문에서 각 예외에 따른 적당한 처리를 하면 된다.
  
  
  
## 루프 취소
  
```
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace Sample09_CancelLoop
{
	class Program
	{
		static void Main()
		{
			var source = new CancellationTokenSource();
			var thread = new Thread(DoCancelableAction);
			thread.Start(source.Token);
			if (Console.ReadKey().Key == ConsoleKey.C)
			{
				Console.Clear();
				Console.WriteLine("press cancel key");
				source.Cancel();
			}
		}

		static void DoCancelableAction(object parameter)
		{
			try
			{
				var option = new ParallelOptions(){ CancellationToken = (CancellationToken)parameter };
				Parallel.ForEach(Enumerable.Range(0, 1000), option, value =>
				{
					Thread.Sleep(100);
					option.CancellationToken.ThrowIfCancellationRequested();
				});
			}
			catch (AggregateException)
			{
				//----- 병렬처리 중에서 예외지만 여기에는 들어오지 않는다
				//----- 취소에 의한 예외는 다르게 처리한다
			}
			catch (OperationCanceledException e)
			{
				Console.WriteLine(e.Message);
			}
		}
	}
}
```
  
- CancellationTokenSource는 예외를 처리하기 위한 오브젝트로 이것을 생성한 토큰에 대해서 취소를 요구할 수 있다.
- ThrowIfCancellationRequested는 아래와 코드와 비슷하므로 호출에 따른 오버헤드가 크지 않다.
  
```
if (token.IsCancellationRequested)
    throw new OperationCanceledException(token);
```
  
- 위 코드의 취소는 수동적 방법으로 이 이외에 콜백이나 대기 핸들을 사용하는 방법도 있다.
   - http://msdn.microsoft.com/ko-kr/library/dd997364.aspx
  
  
  
## 태스크 실행
- 생성과 실행 분리
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample10_TaskStart
{
	class Program
	{
		static void Main()
		{
			var task1 = new Task(() => Console.WriteLine("Task1"));
			var task2 = new Task(() => Console.WriteLine("Task2"));
			var task3 = new Task(() => Console.WriteLine("Task3"));
			task1.Start();
			task2.Start();
			task3.Start();
			Thread.Sleep(1000);
		}
	}
}
```
  
- 위의 코드는 Task 생성 시 생성자 인수로서 비동기로 처리를 델리게이트로 기술하고, 생성된 인스턴스의 Start 매소도를 호출하여 처리를 시작한다.
- Start 메소드는 호출한 스레드를 블럭하지 않는다.
- 생성과 동시에 실행
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample11_TaskFactoryStartNew
{
	class Program
	{
		static void Main()
		{
			var task1 = Task.Factory.StartNew(() => Console.WriteLine("Task1"));
			var task2 = Task.Factory.StartNew(() => Console.WriteLine("Task2"));
			var task3 = Task.Factory.StartNew(() => Console.WriteLine("Task3"));
			Thread.Sleep(1000);
		}
	}
}
```
  
- 암묵적인 태스크
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample12_ParallelInvoke
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine(DebugMessage("Begin"));
			Parallel.Invoke
			(
				() =>
				{
					Thread.Sleep(1000);
					Console.WriteLine(DebugMessage("Task1"));
				},
				() =>
				{
					Thread.Sleep(3000);
					Console.WriteLine(DebugMessage("Task2"));
				},
				() =>
				{
					Thread.Sleep(5000);
					Console.WriteLine(DebugMessage("Task3"));
				}
			);
			Console.WriteLine(DebugMessage("End"));
		}

		static string DebugMessage(string keyword)
		{
			string format = "{0}\n\tTime = {1}\n\tThread ID = {2}\n";
			return string.Format(format, keyword, DateTime.Now.ToLongTimeString(), Thread.CurrentThread.ManagedThreadId);
		}
	}
}
```
  
- Invoke는 인수로 받은 작업을 병렬로 모두 처리할 때까지 호출한 스레드가 블럭된다.
  
  
  
## 완료 대기와 결과 취득
- 시작된 Task가 완료될 때까지 기다리기 위해서는 Wait 메소드를 호출한다. 태스크가 종료될 때까지 호출한 스레드가 블럭된다.
  
```
using System;
using System.Threading.Tasks;

namespace Sample13_TaskWait
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var task = new Task(() => Console.WriteLine("Task is running"));
			task.Start();
			task.Wait();
			Console.WriteLine("End");
		}
	}
}
```
  
- 복수의 태스크의 완료를 대기하고 싶을 때에는 WaitAll을 사용한다.
    - 대기 시간에 타임오버를 설정할 수 있으며, 설정한 경우 시간내에 모두 완료되면 true를 반환한다.
  
```
using System;
using System.Threading.Tasks;

namespace Sample14_TaskWaitAll
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var tasks = new []
			{
				Task.Factory.StartNew(() => Console.WriteLine("Task1 is running")),
				Task.Factory.StartNew(() => Console.WriteLine("Task2 is running")),
				Task.Factory.StartNew(() => Console.WriteLine("Task3 is running")),
			};
			Task.WaitAll(tasks);
			Console.WriteLine("End");
		}
	}
}
```
  
- 복수의 태스크 중 어느 하나가 완료될 때까지만 대기하고 싶은 경우는 WaitAny를 사용한다.
    - 타임아웃 설정이 가능하고, 타임아웃이 발생한 경우 -1을 반환한다.
  
```
using System;
using System.Threading.Tasks;

namespace Sample15_TaskWaitAny
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var tasks = new []
			{
				Task.Factory.StartNew(() => Console.WriteLine("Task1 is running")),
				Task.Factory.StartNew(() => Console.WriteLine("Task2 is running")),
				Task.Factory.StartNew(() => Console.WriteLine("Task3 is running")),
			};
			int index = Task.WaitAny(tasks);
			Console.WriteLine("Index = {0}", index);
			Console.WriteLine("End");
		}
	}
}
```
  
- 결과 얻기
    - Task<TResult> 클래스를 이용한다.
    - Result 메소드는 내부적으로 Wait 메소드를 호출한다.
  
```
using System;
using System.Linq;
using System.Threading.Tasks;

namespace Sample16_TaskResult
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var task = new Task<int>(() => Enumerable.Range(1, 10).Sum());
			task.Start();
			task.Wait();    //--- なくても良い
			Console.WriteLine("Sum = {0}", task.Result);
			Console.WriteLine("End");
		}
	}
}
```
  
  
  
## 태스크 순차 실행
- 태스크1과 태스크2를 순차적으로 실행하고 싶은 경우 ContinueWith 메소드를 사용한다.
  
```
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace Sample17_TaskContinueWith
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Main : Begin");
			var task = new Task<int>(() =>
			{
				Console.WriteLine("Task1 : Begin");
				var sum = Enumerable.Range(1, 10000).Sum();
				Console.WriteLine("Task1 : End");
				return sum;
			});
			task.Start();
			task.ContinueWith(task1 =>
			{
				Console.WriteLine("Task2 : Begin");
				Console.WriteLine("Sum = {0}", task1.Result);
				Console.WriteLine("Task2 : End");
			});
			Console.WriteLine("Main : End");
			Thread.Sleep(1000);
		}
	}
}
```
  
<pre>
결과
Main : Begin
Main : End
Task1 : Begin
Task1 : End
Task2 : Begin
Sum = 50005000
Task2 : End
</pre>
  
    - 위의 두 개의 태스크가 완료될 때까지 이것들을 호출한 스레드는 블럭되지 않는다. 그래서 위의 코드에서는 의도적으로 1초정도 대기를 하고 있다.
    - ContinueWith를 사용하면 앞에 것이 완료된 후 다음 것이 시작되는 것을 보장한다.
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample18_TaskContinueWithMultiple
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var task = new Task(() => Console.WriteLine("Task1"));
			task.Start();
			task.ContinueWith(task1 => Console.WriteLine("Task2"));
			task.ContinueWith(task1 => Console.WriteLine("Task3"));
			Console.WriteLine("End");
			Thread.Sleep(1000);
		}
	}
}
```
  
<pre>
결과
Begin
End
Task1
Task3
Task2
</pre>
  
- 옵션
  
<pre>
옵션	                설명
OnlyOnRanToCompletion	앞의 태스크가 완료까지 실행된 경우에만 다음 태스크를 스케쥴한다.
OnlyOnFaulted	        앞의 태스크에서 예외가 핸들링 된 경우에만 다음 태스크를 스케줄한다.
OnlyOnCanceled	        앞의 태스크가 취소된 경우에만 다음 태스크를 스케줄한다.
</pre>
  
```
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace Sample19_TaskContinuationOptions
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Begin");
			var task = Task<int>.Factory.StartNew(() => Enumerable.Range(1, 10000).Sum());
			task.ContinueWith
			(
				task1 => Console.WriteLine("Success : {0}", task1.Result),
				TaskContinuationOptions.OnlyOnRanToCompletion
			);
			task.ContinueWith
			(
				task1 => Console.WriteLine("Error : {0}", task1.Exception),
				TaskContinuationOptions.OnlyOnFaulted
			);
			task.ContinueWith
			(
				task1 => Console.WriteLine("Task was canceled."),
				TaskContinuationOptions.OnlyOnCanceled
			);
			Console.WriteLine("End");
			Thread.Sleep(1000);
		}
	}
}
```
  
<pre>
결과
Begin
End
Success : 50005000
</pre>
  
  
  
## 태스크 중첩과 자식 태스크
- 중첩 태스크
    - 태스크 A상의 스레드와 다른 스레드에서 태스크 B가 실행된다.
    - 태스크 B를 대기시키는 코드를 명시하지 않으면 태스크 A는 태스크 B의 완료를 기다리지 않고 종료한다.
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample20_NestedTask
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Main : Begin");
			var task1 = Task.Factory.StartNew(() =>
			{
				Console.WriteLine("Task1 : Begin");
				var task2 = Task.Factory.StartNew(() =>
				{
					Console.WriteLine("Task2 : Begin");
					Thread.Sleep(3000);
					Console.WriteLine("Task2 : End");
				});
				//task2.Wait();    //--- コメントの有無で結果が変わります
				Console.WriteLine("Task1 : End");
			});
			task1.Wait();
			Console.WriteLine("Main : End");
		}
	}
}
```
  
<pre>
//----- 결과 (Task2의 대기 없음)
Main : Begin
Task1 : Begin
Task1 : End
Main : End
Task2 : Begin

//----- 결과 (Task2의 대기 있음)
Main : Begin
Task1 : Begin
Task2 : Begin
Task2 : End
Task1 : End
Main : End
</pre>
  
- 부모자식 관계를 가진 태스크
    - 채스크에 부모자식 관계를 가지기 위해서는 태스크 생성 시에 TaskCreationOptions.AttachedToParent 를 지정한다.
    - 태스크가 실행을 완료할 때까지 부모가 완료된 이후에 자식이 완료된다.
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample21_ParentChildTask
{
	class Program
	{
		static void Main()
		{
			Console.WriteLine("Main : Begin");
			var task1 = Task.Factory.StartNew(() =>
			{
				Console.WriteLine("Task1 : Begin");
				Task.Factory.StartNew(() =>
				{
					Console.WriteLine("Task2 : Begin");
					Thread.Sleep(3000);
					Console.WriteLine("Task2 : End");
				}, TaskCreationOptions.AttachedToParent);
				Console.WriteLine("Task1 : End");
			});
			task1.Wait();
			Console.WriteLine("Main : End");
		}
	}
}
```
  
<pre>
Main : Begin
Task1 : Begin
Task1 : End
Task2 : Begin
Task2 : End
Main : End
</pre>
  
  
  
## 태스크의 예외 처리
- 예외 포착
    - 태스크 중에서 처리되지 않은 예외는 태스크 자신이 포착하여 컬렉션으로 보존한다.
    - Wait 혹은 Result 프로퍼티가 실행되면 이 멤버들에서 System.AggregateException가 스로우된다.
    - 태스크가 포착한 예외는 스로우된 AggregateException의 InnerExceptions 프로퍼티에서 취득할 수 있다.
  
```
using System;
using System.Threading.Tasks;

namespace Sample22_TaskException
{
	class Program
	{
		static void Main()
		{
			var task = Task.Factory.StartNew(() =>
			{
				throw new Exception("Test Exception");
			});
			try
			{
				task.Wait();
			}
			catch (AggregateException exception)
			{
				foreach (var inner in exception.InnerExceptions)
					Console.WriteLine(inner.Message);
			}
		}
	}
}
```
  
<pre>
결과
Test Exception
</pre>
  
- 자식 태스크에서 발생한 예외 처리
  
```
using System;
using System.Threading.Tasks;

namespace Sample23_NestedException
{
	class Program
	{
		static void Main()
		{
			var task = Task.Factory.StartNew(() =>
			{
				Task.Factory.StartNew(() =>
				{
					throw new Exception("Task2 : Exception");
				}, TaskCreationOptions.AttachedToParent);
				throw new InvalidOperationException("Task1 : Exception");
			});
			try
			{
				task.Wait();
			}
			catch (AggregateException exception)
			{
				foreach (var inner in exception.InnerExceptions)
				{
					Console.WriteLine(inner.Message);
					Console.WriteLine("Type : {0}", inner.GetType());
				}
			}
		}
	}
}
```
  
<pre>
결과
Task1 : Exception
Type : System.InvalidOperationException
하나 이상의 에러가 발생.
Type : System.AggregateException
</pre>
  
- 자식 태스크에서 발생한 예외 검출시에 Flatten 메소드 사용
  
```
using System;
using System.Threading.Tasks;

namespace Sample24_ExceptionFlatten
{
	class Program
	{
		static void Main()
		{
			var task = Task.Factory.StartNew(() =>
			{
				Task.Factory.StartNew(() =>
				{
					throw new Exception("Task2 : Exception");
				}, TaskCreationOptions.AttachedToParent);
				throw new InvalidOperationException("Task1 : Exception");
			});
			try
			{
				task.Wait();
			}
			catch (AggregateException exception)
			{
				foreach (var inner in exception.Flatten().InnerExceptions)
				{
					Console.WriteLine(inner.Message);
					Console.WriteLine("Type : {0}", inner.GetType());
				}
			}
		}
	}
}
```
  
<pre>
결과
Task1 : Exception
Type : System.InvalidOperationException
Task2 : Exception
Type : System.Exception
</pre>
  
- 미처리 예외
    - 아래의 상황에서 예외가 발생하면 확인할 수 없다.
    - Task.Wait 메소드를 호출 하지 않는다.
    - Task<TResult>.Result 프로퍼티를 호출 하지 않는다.
    - Task.Exception 프로퍼티를 호출하지 않는다.
    - 그러나 방법은 있다. 태스크 인스턴스가 가베지 컬렉션에 의해 회수될 때 Task.Finalize 메소드는 자신의 예외가 확인되고 있는지 확인한다. 확인되지 않았다고 판단된 경우 Finalize 메소드에서 AggregateException가 스로우된다. Finalize 메소드는 CLR 전용 스레드인 Finalizer 스레드 상에서 실행되기 때문에 그 예외는 포착할수 없고 프로세스가 즉각 종료된다.
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample25_UnobservedTaskException
{
	class Program
	{
		static void Main()
		{
			TaskScheduler.UnobservedTaskException += TaskScheduler_UnobservedTaskException;
			Task.Factory.StartNew(() => { throw new InvalidOperationException("Task1"); });
			Task.Factory.StartNew(() => { throw new InvalidCastException("Task2"); });
			Thread.Sleep(300);                //--- 태스크 종류를 대기
			GC.Collect();                     //--- 태스크 인스턴스를 회수
			GC.WaitForPendingFinalizers();    //--- Finalize를 강제적으로 호출한다
			Console.WriteLine("End");
		}

		static void TaskScheduler_UnobservedTaskException(object sender, UnobservedTaskExceptionEventArgs e)
		{
			foreach (var inner in e.Exception.Flatten().InnerExceptions)
			{
				Console.WriteLine(inner.Message);
				Console.WriteLine("Type : {0}", inner.GetType());
			}
			e.SetObserved();    //--- 処理済みとしてマークする
		}
	}
}
```
  
<pre>
결과
Task1
Type : System.InvalidOperationException
Task2
Type : System.InvalidCastException
End
</pre>
  
  
  
## 태스크 취소
- 시작과 동시에 취소
  
```
using System;
using System.Threading;
using System.Threading.Tasks;

namespace Sample26_CancelTaskStart
{
	class Program
	{
		static void Main()
		{
			var source  = new CancellationTokenSource();
			var task    = new Task(() => Console.WriteLine("태스크가 실행 되었다."), source.Token);
			task.Start();
			source.Cancel();
			try
			{
				task.Wait();
			}
			catch (AggregateException exception)
			{
				foreach (var inner in exception.InnerExceptions)
				{
					Console.WriteLine(inner.Message);
					Console.WriteLine("Type : {0}", inner.GetType());
				}
			}
		}
	}
}
```
  
<pre>
결과
태스크가 취소 되었다.
Type : System.Threading.Tasks.TaskCanceledException
</pre>
  
    - 태스크의 시작과 취소는 비동기로 실행되기 때문의 위의 코드는 시작도 못하고 바로 취소 되었다.
    - 만약 시작 이후 취소까지 대기 타임을 주면 취소 되지 않고 실행된다.
- 실행 중에 취소
  
```
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace Sample27_CancelTaskByPolling
{
	class Program
	{
		static void Main()
		{
			var source  = new CancellationTokenSource();
			var token   = source.Token;
			var task    = new Task(() =>
			{
				Console.WriteLine("Task : Begin");
				foreach (var item in Enumerable.Range(0, 10))
				{
					token.ThrowIfCancellationRequested();
					Thread.Sleep(100);
				}
				Console.WriteLine("Task : End");
			});
			task.Start();
			Thread.Sleep(300);    //--- 태스크가 실행되도록 한다.
			source.Cancel();
			try
			{
				task.Wait();
			}
			catch (AggregateException exception)
			{
				foreach (var inner in exception.InnerExceptions)
				{
					Console.WriteLine(inner.Message);
					Console.WriteLine("Type : {0}", inner.GetType());
				}
			}
		}
	}
}
```
  
<pre>
결과
Task : Begin
조작은 취소 되었다.
Type : System.OperationCanceledException
</pre>
  
  
  
## UI 컴포넌트 조작
- TPL에는 스케쥴이라는 개념이 있으며 이것이 태스크의 실행 순서를 정하고 스레드로 실행되도록 한다.TPL 표준에서는 스레드 풀 태스크 스케쥴러와 동기 컨텍스트 스케쥴러가 있다.
- TPL의 기본은 스레드 풀 스케쥴러 이며 정적인 TaskScheduler.Default 프로퍼티에서 얻을 수 있다. 스레드 풀 스케줄러는 그 이름대로 태스크를 스레드 풀의 워커스레드로 등록하여 처리한다.
- 동기 컨텍스트 스케쥴러는 정적인 TaskScheduler.FromCurrentSynchronizationContext 메소드에서 얻을 수 있다. 이 스케쥴러는 Windows Forms나 WPF 등의 GUI 애플리케이션에 주로 사용되며, 버턴이나 메뉴 등의 UI 컴포넌트를 태스크 상에서 갱신하도록 태스크를 애플리케이션 UI 스레드에 등록하여 처리하도록 한다. 스레드 풀은 일정 사용할 수 없다.
- 동기 컨텍스트 스케쥴러 사용
  
```
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace Sample28_SynchronizationContextTaskScheduler
{
	public partial class MainForm : Form
	{
		public MainForm()
		{
			this.InitializeComponent();
			this.button.Click += this.Button_Click;
		}

		private void Button_Click(object sender, EventArgs e)
		{
			this.button.Enabled = false;
			Task.Factory.StartNew(() =>
			{
				Thread.Sleep(3000); //---- Do Somothing
			})
			.ContinueWith(parent =>
			{
				this.button.Enabled = true;
			}, TaskScheduler.FromCurrentSynchronizationContext());
		}
	}
}
```
  
  
  
## TaskFactory.FromAsync
- 출처: http://kyeongkyun.tistory.com/85
- IAsyncResult 패턴을 Task로 변환 할 때 사용하면 좋다
- IAsyncResult 기반의 비동기 연산인 Dns.BeginGtHostAddress를 Task로 변환
  
```
static void Main() {
  Task<IPAddress[]> task =
	Task<IPAddress[]>.Factory.FromAsync(
	  Dns.BeginGetHostAddresses,
	  Dns.EndGetHostAddresses,
	  "http://www.microsoft.com", null);
  ...
}
```
  
- 내부적으로 FromAsync 메서드는 스레드 풀을 활용하는 TaskCompletionSource의 예제와 비슷한 방법으로 구현
  
```
static Task<IPAddress[]> GetHostAddressesAsTask(
  string hostNameOrAddress) {

  var tcs = new TaskCompletionSource<IPAddress[]>();
  Dns.BeginGetHostAddresses(hostNameOrAddress, iar => {
	try {
	  tcs.SetResult(Dns.EndGetHostAddresses(iar)); }
	catch(Exception exc) { tcs.SetException(exc); }
  }, null);
  return tcs.Task;
}
```
  
- (일본어)TPL과 기존 .NET 비동기 프로그래밍
    - http://msdn.microsoft.com/ja-jp/library/vstudio/dd997423.aspx
  
  
  
## Task.Run
- 닷넷 프레임워크 4.5 에서 추가된 것
  
```
//.Net framework 4.5에 추가된 메소드로
//Task.Factory.StartNew 래핑한다.
Task.Run(() => { /* Something */ });
```
  
<img src="https://drive.google.com/file/d/0B1EFZOfsV3gqMjkxcEN5TElsRFk">
  
- http://csharp.keicode.com/basic/async-how-to-write.php
  
  
  
## 복수의 task를 동시에 실행. Task.WhenAll(async/await), Task.WaitAll
- [WhenAll](http://msdn.microsoft.com/ko-kr/library/system.threading.tasks.task.whenall.aspx)
  
```
await Task.WhenAll( redisObj.SetField("ID", userData.ID),
                     redisObj.SetField("PW", userData.PW),
                     redisObj.SetField("AuthToken", userData.AuthToken),
                     redisObj.SetField("UnqiueNumber", userData.UnqiueNumber) );
```
  
- [WaitAll](http://msdn.microsoft.com/ko-kr/library/dd270695.aspx )
  
```
Task.WaitAll( redisObj.SetField("ID", userData.ID),
              redisObj.SetField("PW", userData.PW),
              redisObj.SetField("AuthToken", userData.AuthToken),
              redisObj.SetField("UnqiueNumber", userData.UnqiueNumber) );
```
  