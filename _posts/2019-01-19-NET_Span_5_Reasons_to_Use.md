---
layout: post
title: Span <T>를 사용해야 할 5가지 이유
published: true
categories: [.NET]
tags: .net c# .netcore span
---
[출처](https://qiita.com/GlassGrass/items/cea3c6f91413c3582b5f )  

## 소개: Hello World
런타임이 .Net Core 2.1 이후라면 표준으로 사용할 수 있다.  
그렇지 않으면 Nuget에서 System.Memory 라는 패키지를 넣자.  

그리고, 언어는 C# 7.2 이상이 필요하다.   

  
  
## Span<T>라는게 뭐야?
A. 우선, 배열 같은 것이라고 생각해도 좋다.  

정확하게 말한다면, 배열의 일부분을 가리키는 뷰이다.  
다음 코드에서는 array의 뷰로 span을 만들고 있다.  
```
using System;

public static class Program
{
    public static void Main()
    {
        var array = new int[]{0, 1, 2, 3, 4, 5, 6};
        var span = new Span<int>(array);
        Console.WriteLine(span[3]); // result: 3
        Console.WriteLine(span.Slice(2)[3]); // result: 5
    }
}
```
  
<br/>
  
Span<T>의 읽기 전용 버전으로 ReadOnlySpan<T>도 준비되어 있다.  
```
static class HogeClass
{
    public static void SomeMethod1(Span<int> span)
    {
        Console.WriteLine(span[0]); // OK
        span[0] = 3; // OK
    }

    public static void SomeMethod2(ReadOnlySpan<int> span)
    {
        Console.WriteLine(span[0]); // OK
        span[0] = 3; // NG
    }
}
```
  
Span<T>, ReadOnlySpan<T>를 만드는 기본적인 경로는 다음 그림과 같다. 위의 3가지 타입에서 직접 ReadOnlySpan<T>을 만들 수 있지만 그림이 어수선해지기 때문에 여기에서는 생략하고 있다. ReadOnlySpan<char>에 한하여 string에서 직접 생성 할 수 있다.  

![](https://camo.qiitausercontent.com/db743820020557e8e50585cc46632d7874d35984/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3133303838352f38636132633562342d393863332d396463322d653461322d6665373865343536303237662e706e67)
  
그림을 보면 알 수 있듯이, Span<T>이 다루는 배열이란 단순히 System.Array 파생 타입뿐만 아니라 스택 상의 배열과 관리되지 않는 힙을 포함한 낮은 수준의 선형 데이터 구조 전반을 가리킨다.  

또한 Span<T> / ReadOnlySpan<T>는 ref 구조체라는 특별한 구조체이다.  
간단하게 말하면, 객체의 실체가 반드시 스택에 놓여지는 것이 보장 되는 타입이다. 이것이 어떻게 효과를 낳는 지에 대해서는 후술한다.

  
<br/>
  

## Span<T>를 사용해야 하는 이유
### 1. 잘 만들어진 배열 view
종래 .Net의 배열 뷰라고 하면 System.ArraySegment<T> 타입이었다.  
Span<T>에는 ArraySegment<T>에 비해 다음과 같은 장점이 있다.  
- 성능이 좋다.
- 읽기 전용 버전(ReadOnlySpan<T>)가 준비되어 있다.
- System.Array뿐만 아니라 스택 배열과 관리되지 않는 힙에 대해서도 이용 가능하다.
- T가 관리되지 않는 타입일 때, MemoryMarshal에 따라 타입을 넘는 유연한 읽고 쓰기가 가능하다.
- IList<T>으로 캐스팅 하지 않아도 인덱서를 사용할 수 있다.
  
  
대충 이것만으로도 Span<T>의 존재 의의가 전해질 것이다.   
물론 ArraySegment<T>은 할 수 있지만 Span<T>는 할 수 없는 경우도 있지만, Span<T>쪽이 뛰어난 부분이 많다.  
  

### 2. No more unsafe
C#과 같은 객체 지향 언어에서는 외부에서 본 행동을 인터페이스로 정의하고 구현을 은폐함으로써 복잡한 구현에서 추상화된 기능을 꺼내왔다. 이러한 객체 지향의 실현 기구는 어느 정도의 계산 능력을 필요로 하고 있지만, 오늘날의 개발은 거의 필수라고 말해도 좋을 정도로 성공을 거두고 있다.  

그런데 C#은 상호 운용성이나 성능 이라든지에 적당히 신경을 쓰고 있는 언어이므로, 부분적으로 제한을 걸면서 낮은 수준의 데이터 구조를 지원하고 있다. 즉 System.Array, 스택 배열, 관리되지 않는 힙 이라는 세 종류의 배열이 있다.  

기존의 안전한 컨텍스트에서 사용할 수 있는 것은 System.Array뿐으로 다른 2개의 이용에는 unsafe가 필요했다.  3종의 배열은 모두 첫 번째 요소에 대한 참조와 배열의 길이를 가지며, 내부적으로 메모리에 연속하고, 색인 사용이 가능하다는 그야말로 추상화 할 것 같은 공통된 기능을 가지지만, 포인터 없이는 실현 될 수 없다. 라이브러리라면 몰라도 응용 프로그램 코드를 안전하지 않은 컨텍스트로 쓴다는 것은 마음이 내키지 않는다.  

그러던 Span<T>이 구현 되었다. 앞서 언급했듯이, Span<T>은 3종의 배열 구조에서도 만들 수 있을뿐만 아니라 같게 다룬다. 인터페이스 타입과는 다르지만, "배열 타입"으로 기대되는 행동을 멋지게 추상화 되어 있다고 할 수 있겠다.  
  
  
### 3. The type is the document, the type is the contract
내가 생각하는 정적 타이핑의 가장 큰 장점은 형식 자체가 문서화 되고, 또한 계약이 되는 것이다.  
타입이 명시된 코드는 컴파일 시에 검증된 일종의 코딩 실수는 논리적 보증에서 검출된다.  
예를 들어, IReadOnlyList<T> 타입 개체의 인덱서에 값을 set 하려고 하면 튕겨진다.  정적 타입 언어로 프로그래밍을 하는데 이 장점을 살리는 코드를 작성하는 것은 소중히 하고 싶다는 지침이다.  

기존 배열에 대해서 타입에 의한 계약을 베푸는 수단은 부족했다.   
예를 들어 다음의 코드를 보자.  
```
public class StreamReadingBuffer
{
    public int CurrentPosition { get; private set; }

    public byte[] Buffer => _buffer;
    private readonly byte[] _buffer;

    // ～～～
}
```
  
이 클래스는 이름에서 알 수 있듯이 바이너리 스트림 읽기를 효율화 하기 위해 버퍼를 감싼 것이다.  
현재의 읽기 버퍼를 Buffer 속성으로 공개하고 있지만 byte[] 형식은 내용의 변경을 방지 할 수 없다.  
게다가 보지 않으면 안되는 위치도 CurrentPosition 속성으로 분리되어 버렸다.   
보통 이러한 경우에는 원시 형식 대신 IReadOnlyList<byte> 등의 형태로 읽기 전용 제약을 거는 것을 권장되지만, 일부러 배열 형을 사용하고 있기 때문에 상상할 수 있듯이, 이 녀석은 꽤 성능에 신경을 쓴 케이스로 사용되는 것을 상정하고 있다. 불필요한 인터페이스를 사이에 두어서 실행 속도를 악화시키는 시그니쳐는 안전하다 해도 미움 받는다.  

Span<T> / ReadOnlySpan<T>이 도입 된 것으로,이 같은 장면은 효율성과 안전성을 양립 할 수 있게 되었다.  
```
public class StreamReadingBuffer
{
    private int _currentPosition;

    public ReadOnlySpan<byte> Buffer => _buffer.Slice(_currentPosition);
    private readonly byte[] _buffer;

    // ～～～
}
```
  
위의 코드에서는 Buffer의 내용을 클래스의 사용자가 다시 작성할 수 없으며, 수신 버퍼 자체가 현재 위치에 따라서 분리해 낸 것으로 되어 있다. 이 변경에 따른 배열에 대한 오버 헤드도 극히 소량이다.  
읽기 전용인 것과 봐야할 장소가 확보된 메모리 전체가 아닌 일부라는 것이 ReadOnlySpan<T>로 있다는 사실에 의해 코드에서 표현되고 있는 것이다.  
  
  
### 4. 그래도 나는 바이너리를 읽고 싶다
보통의 배열에 대해서도 그 표현력에 따라 역할을 가질 Span<T>이지만 이 진가는 T가 관리되지 않는 형식 일 때 발휘된다.  

관리되지 않는 형식은 .Net의 관리 참조를 포함하지 않는 형태이다.  
가비지 컬렉터가 신경 쓰지 않는 타입 또는 C로 동등한 구조체를 만들 수 있는 형태라고 생각해도 좋다.  

모든 비 관리 타입 객체는 등가인 바이트 배열을 생각할 수 있다.  
Span<T>은 MemoryMarshal을 사용하여 임의의 관리 되지 않는 형식 간에 변환 할 수 있다.  
```
using  System.Runtime.InteropServices;

// float 배열을 int 배열로서 읽고 쓸 수 있도록 한다. 
public void Foo(Span<float> bufferAsFloat)
{
    Span<int> bufferAsInt = MemoryMarshal.Cast<float, int>(bufferAsFloat);
    // ～～～
}
```
  
이 기능을 사용하려면 최대의 유스 케이스는 바이너리 스트림의 읽고 쓰기이다.  
독자적인(또는 독자적인 아닌 특수한) 프로토콜과 파일 형식을 구현할 때 구조체를 정의 해두면 빠르고 간단한 읽고 쓰기가 가능하게 된다. 예를 들어, 다음 코드는 PortableExecutable 파일의 헤더를 읽는 코드이다.  

< PE 파일 헤더를 읽는다 >  
```
using System;
using System.Runtime.InteropServices;

// DosHeader, ImageFileHeader, ImageOptionalHeaderなどは割愛
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct NtHeader
{
    public readonly uint Signature;
    public readonly ImageFileHeader FileHeader;
    public readonly ImageOptionalHeader OptionalHeader;
}

public static class Sample
{
    public static NtHeader ReadHeader(byte[] buffer)
    {
        var dosHeader = MemoryMarshal.Cast<byte, DosHeader>(buffer)[0];
        return MemoryMarshal.Cast<byte, NtHeader>(buffer.AsSpan(dosHeader.e_lfanew))[0];
    }
}
```
  
이른바 C++의 reinterpret_cast 같은 동작이지만, 이것을 손쉽게 할 수 있게 된 셈이다.  
  
  
### 5. API 충분
ArraySegment<T>은 BCL 내부에서의 대응도 짧고, 표준 API로 다루기에 미묘했다.  
어떤 타입이 사용될지 여부에 대해 API의 충족은 중요한 요소이다.  
전달도 받을 수도 없다면 결국 직접 변환이 필요하게 되고, 볼 기회도 훨씬 줄어들 것이다.  

Span<T>의 도입에서, 다음의 타입 등에 대응이 담겨 있다.  
- 기본 숫자 형식
- System.BitConverter
- System.IO.Stream
- Sytem.Text.Encoding
  
즉, 지금까지 낮은 수준 용도로 배열을 받는 API에 상당 부분 Span<T>이 대응된 것이다.   
전술 한 바와 같이 Span<T>는 stackalloc도 사용할 수 있으므로 힙을 사용하지 않고 스트림을 읽을 수 있다.  
  
  
  
## Span<T>를 사용하면 안 되는 3개의 케이스
뭐, 여기까지 Span<T> 선전을 해 왔지만, 모든 경우에 최적의 솔루션이라는 것은 없다.   
오히려 어떤 면에서의 편리성을 추구한 결과, 다른 방면에서는 불편한 형태가 된다.  
여기에서는 이 성격에서 Span<T> 사용이 적합하지 않거나 사용할 수 없는 케이스를 소개한다.  

### 클래스의 멤버로 사용할 수 없던데?
**A. 스펙이다.**
  
앞서 언급했듯이 Span<T>은 ref 구조체라는 타입이다.  
ref 구조체의 객체는 반드시 스택에 놓여 있는 것을 보증하고 있다.   
반복하면 힙에 실릴 수 있는 것은 일절 할 수 없다는 것이다.  

< Span이 할 수 없는 것 >  
```
public class Hoge
{
    private Span<object> _span; // NG: ref 구조체 타입의 필드로 되지 앟는다.

    public static Span<int> Foo()
    {
        Span<int> span = stackalloc int[16];
        Bar(span); // NG: 제널릭 타입인수로 할 수 없다.
        var obj = (object)span; // NG: object로 업캐스트 할 수 없다
        Func<int> baz = () => span[0]; // NG: 델리게이트 캡쳐할 수 없다.

        // True. 업캐스트 할 수 없지만 내부적으로는 object의 파생 타입.
        // 어디까지나 C# 컴파일러가 정적 검증으로 금지하고 있을뿐이므로 당연하다면 당연하다.
        Console.WriteLine($"Span<T> is a subtype of object: {typeof(Span<>).IsSubclassOf(typeof(object))}");

        return span; // NG: stackalloc은 메소드 영역에서 나오면 죽으므로 반환값으로 할 수 없다.
    }

    public static void Bar<T>(T value){}
}

public ref struct Fuga : IEnumerable<int> // NG: 인터페이스를 구현할 수 없다
{
}
```
  
당연하지만, 클래스와 인터페이스를 통해 객체 지향을 실현하는 C#에서 이것은 매우 엄격한 제한이다.   
여기서 제한에 걸리는 용도라면, 배열을 사용하거나 System.Memory<T>라는 형식을 사용하자.  
  
  
## IList<T>/ IReadOnlyList<T>/ ICollection<T>/ IEnumerable<T>로 충분?
만약 그렇게 생각한다면, 대개의 경우 그 감각이 옳은 것 같다.   
이미 언급 한 바와 같이, Span<T>는 저수준 프로그래밍을 위해 존재한다.  
대규모 응용 프로그램도 적당히 쉽게 보수성 좋게 쓸 수 있는 C#에서 원시 바이너리에 주목하는 케이스는 결코 많지 않다.  
실체는 List<T>가 되는것을 사용으로 두어서 적절한 인터페이스를 공개하도록 하는 것이 .Net 타입 시스템에서 솔직하고, 사용 측에 매우 편리하고 직관적인 것이 사상의 대부분이다.   
성능이 매우 중요하다든가, 레거시 프로토콜을 구현하고 싶다든가, 명확한 이유가 없다면 Span<T>을 쓸 필요는 없다.  
  
  
## 마지막
Span<T>은 어쨌든 사용하는 수준의 것이 아니지만, 속도가 중요한 프로그래밍에 확실히 힘을 실어 준다.   
만약 당신이 I/O나 통신의 핵심 라이브러리를 쓰는 일에 관련하고 있다면, 꼭 사용하기 바란다.  

