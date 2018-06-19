---
layout: post
title: .NET Core 2.0의 Random과 이전 버전(닷넷프레임워크 포함)과의 차이
published: true
categories: [.NET]
tags: c# .net random
---
.NET Core 2.0 이전 버전 및 닷넷프레임워크의 Random 클래스는 생성 시 직접 seed 값을 주지 않으면 Environment.TickCount 를 사용한다.    
그래서 동시에 복수의 Random 클래스를 생성하면 seed 값이 같아서 복수의 Random 인스턴스에서 값은 패턴의 난수가 발생한다.  
  
.NET Core 2.0 에서는 이런 문제가 수정 되었다.   
   
     
```
class Program
{
	static void Main(string[] args)
	{
		Console.WriteLine("Hello World!");

		for (var i = 0; i < 10; i++)
		{
			Console.WriteLine(new Random().Next());
		}
	}
}
```   
위 코드를 .NET Core 2.0 과 .NET Framework 4.7 에서 실행하면 각각 아래와 같다.  
![](/images/2018/2018-02-25-01.PNG)     
  