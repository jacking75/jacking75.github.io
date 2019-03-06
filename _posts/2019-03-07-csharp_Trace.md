---
layout: post
title: C# - Trace
published: true
categories: [.NET]
tags: c# net trace
---
## Trace 클래스
- 코드 실행을 추적하는 데 필요한 메서드 및 속성 집합을 제공
- 네임스페이스: System.Diagnostics
- 어셈블리: System(system.dll)
- Trace 클래스의 속성 및 메서드를 사용하여 릴리스 빌드를 측정 할 수 있다. 이렇게 하면 실제 설정에서 실행되는 응용 프로그램의 상태를 모니터링할 수 있다. 추적 기능을 이용하여 실행 중인 시스템에 영향을 주지 않고 문제를 확인하여 해결할 수 있다.
    - C#에서 추적을 활성화하려면 코드를 컴파일할 때 /d:TRACE 플래그를 컴파일러 명령줄에 추가하거나, #define TRACE를 파일 맨 위에 추가한다. 
- BooleanSwitch 및 TraceSwitch 클래스는 추적 출력을 동적으로 제어하는 방법을 제공한다. 응용 프로그램을 다시 컴파일하지 않고도 이들 스위치 값을 수정할 수 있다. 
- Listeners 컬렉션에서 TraceListener 인스턴스를 추가하거나 제거하여 추적 출력 대상을 사용자 지정할 수 있다. 기본적으로 DefaultTraceListener 클래스를 사용하여 추적 출력을 내보낸다.
- Trace 클래스는 Indent 및 IndentSize 수준과 각 쓰기 직후의 AutoFlush 호출 여부를 가져오거나 설정하는 속성을 제공한다.
    - Trace의 AutoFlush 및 IndentSize를 설정하려면 응용 프로그램 이름과 일치하는 구성 파일을 편집한다. 구성 파일의 형식은 다음 예제와 같아야 한다.
  
```
<configuration>
  <system.diagnostics>
	<trace autoflush="false" indentsize="3" />
  </system.diagnostics>
</configuration>
```
  
  
  
## TraceSwitch 클래스
- 코드를 다시 컴파일하지 않고 추적 및 디버그 출력을 제어하는 여러 수준의 스위치를 제공한다
- 추적 스위치를 사용하여 메시지를 중요도에 따라 필터링할 수 있다. 
- TraceSwitch 클래스는 스위치의 수준을 테스트하는 TraceError, TraceWarning, TraceInfo 및 TraceVerbose 속성을 제공한다. 
    - Level 속성은 스위치의 TraceLevel을 가져오거나 설정한다.
- 응용 프로그램 구성 파일을 통해 TraceSwitch의 수준을 설정한 다음 응용 프로그램에서 구성된 TraceSwitch 수준을 사용할 수 있다. 또는 코드에서 TraceSwitch를 만들고 수준을 직접 설정하여 코드의 특정 섹션을 조정할 수 있다.
- TraceSwitch를 구성하려면 응용 프로그램의 이름에 해당하는 구성 파일을 편집한다. 이 파일에서 스위치를 추가 또는 제거하거나, 스위치 값을 설정하거나, 응용 프로그램에서 이전에 설정한 모든 스위치를 지울 수 있다. 구성 파일의 형식은 다음 예제와 같아야 합니다.
  
```
<configuration>
  <system.diagnostics>
	<switches>
	  <add name="mySwitch" value="1" />
	</switches>
  </system.diagnostics>
</configuration>
```
  
```
private static TraceSwitch appSwitch = new TraceSwitch("mySwitch", "Switch in config file");

public static void Main(string[] args) 
{
	//...
	Console.WriteLine("Trace switch {0} configured as {1}", 
	appSwitch.DisplayName, appSwitch.Level.ToString());
	if (appSwitch.TraceError)
	{
		//...
	}
}
```
  
- 다음 코드 예제에서는 새 TraceSwitch를 만들고 해당 스위치를 사용하여 오류 메시지의 출력 여부를 결정한다
  
```
//Class-level declaration.
 /* Create a TraceSwitch to use in the entire application.*/
 static TraceSwitch mySwitch = new TraceSwitch("General", "Entire Application");

 static public void MyMethod() {
	// Write the message if the TraceSwitch level is set to Error or higher.
	if(mySwitch.TraceError)
	   Console.WriteLine("My error message.");

	// Write the message if the TraceSwitch level is set to Verbose.
	if(mySwitch.TraceVerbose)
	   Console.WriteLine("My second error message.");
 }

 public static void Main(string[] args) {
	// Run the method that prints error messages based on the switch level.
	MyMethod();
 }
```
  
- 성능을 향상시키려면 클래스에서 TraceSwitch 멤버를 static으로 설정합니다.
  
  
  
## TraceLevel 열거형
- Debug, Trace 및 TraceSwitch 클래스에 출력할 메시지를 지정한다
- public enum TraceLevel
  
```
<configuration>
	 <system.diagnostics>
		<switches>
		   <add name="mySwitch" value="4" />
		</switches>
	 </system.diagnostics>
 </configuration>
```
  
<pre>
Off (0)		추적 및 디버깅 메시지를 출력하지 않습니다.
Error (1)	오류 처리 메시지를 출력합니다.
Warning (2)	경고 및 오류 처리 메시지를 출력합니다.
Info (3)	정보 메시지, 경고 및 오류 처리 메시지를 출력합니다.
Verbose (4)	디버깅 및 추적 메시지를 모두 출력합니다.
</pre>
  
  
  
## TraceListener의 종류
- TextWriterTraceListener
- EventLogTraceListener
- DefaultTraceListener
- ConsoleTraceListener
- DelimitedListTraceListener
- XmlWriterTraceListener
  
  
  
## BooleanSwitch
- 디버그 및 트래이스 출력 유무를 제어 할 수 있다.
  
```
<configuration>
  <system.diagnostics>
	<switches>
	  <add name="mySwitch" value="1"/>
	</switches>
  </system.diagnostics>
</configuration>
```
  
- 코드로 제어
  
```
BooleanSwitch mySwitch = new BooleanSwitch("mySwitch", "Sample of BooleanSwitch");
if ( mySwitch.Enabled )
	 Trace.WriteLine("Switch on");
```
  
  
  
## 예제. 프로그램 실행의 시작과 끝을 보기
- 이 예제에서는 Indent 및 Unindent를 사용하여 추적 출력을 구분한다
  
```
static int Main(string[] args)
{
   Trace.Listeners.Add(new TextWriterTraceListener(Console.Out));
   Trace.AutoFlush = true;
   Trace.Indent();
   Trace.WriteLine("Entering Main");
   Console.WriteLine("Hello World.");
   Trace.WriteLine("Exiting Main"); 
   Trace.Unindent();
   return 0;
}
```
  
  
  
## 예제. 파일로 남기기
- 출처: http://blog.naver.com/manylee0/110010057717
  
```
// Global.cs
using System;
using System.Diagnostics;

namespace SFSecsUnitTest
{
	static class Global
	{
		static TraceSource debugTraceSource;
		static Global()
		{
			Trace.AutoFlush = true; // turn on auto flush

			debugTraceSource = new TraceSource("SFSecsUnitTest"); 
			debugTraceSource.Switch.Level = SourceLevels.All;

			TraceListener listener = new TextWriterTraceListener(
										@"c:\SFSecsUnitTest.trace", "TextWriterListener");
			listener.TraceOutputOptions = TraceOptions.DateTime |
                                                                TraceOptions.Callstack |
                                                                TraceOptions.ThreadId |
                                                                TraceOptions.Timestamp;

			debugTraceSource.Listeners.Add(listener);
		}
		public static TraceSource DebugTraceSource
		{
			get
			{
				return debugTraceSource;
			}
		}
	}
}

// GlobalTest.cs (unit test)

using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Diagnostics;

namespace SFSecsUnitTest
{
	[TestClass]
	public class GlobalTest
	{
		[TestMethod]
		public void TestMethod1()
		{
			Global.DebugTraceSource.TraceEvent(TraceEventType.Verbose, 1,"GlobalTest.TestMethod1() - Enter");

		}
	}
}
```
  
  
  
## 닷넷 네트워크 라이브러리의 트레이스 기능
- http://www.sysnet.pe.kr/2/0/1074
  