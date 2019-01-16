---
layout: post
title: Span <T>이용에 따른 최적화
published: true
categories: [.NET]
tags: .net c# .netcore span
---
[출처](https://ufcpp.net/blog/2018/12/spanify/ )  

## 힙 사용량 절감
Span<T> 를 사용하면 빠르게 되는 이유는 간단하게 힙 사용량을 줄아가 때문이다.  
- string.Substring 등으로 새로운 문자열을 만들어 않아도 된다.
- stackallock욿 임시 버퍼에 힙을 사용하지 않는다
- 네이티브 메모리를 직접 읽을 수 있게함으로써 관리되는 배열에 복사하지 않는다
  
모두 unsafe 코드 포인터를 사용하면 지금까지도 충분히 실현 될 수 있었던 것입니다.  
그러나 안전성과 생산성을 희생한 코드는 쓰는 것도 사용하는 것도 신경을 써야 하기 때문에 대규모로 도입하기 어렵고, GC의 제한도 있고, Span<T> 없이는 어려운 최적화이다.  

Span<T>도 여러가지 제한이 걸린 특수한 타입( ref struct )이지만, 여전히 포인터 보다는 적용 가능한 범위가 넓다.  
  
  
## Substring
.NET의 string.Substring은 새로운 string 타입 인스턴스(물론 힙을 사용)를 만들고 이것을 반환 값으로 반환한다.  

섣불른 가상 호출 증가보다는 불필요한 힙을 사용하는 쪽이 고속 이므로 이렇게 만들지만, Span<char>이 있다면 힙을 사용하지 않고 비슷하게 할 수 있다.  

Substring뿐만 아니라 문자열 조작에 관련된 부분은 Span<T>의 혜택을 받고 있으며 배 이상 빨라진 메소드도 몇 개 있다.  
  
  
## stackalloc
극히 짧은 범위에서 작은 데이터를 가지는 임시 버퍼를 필요로 하는 것은 상당히 있다. 이런 때, 지금까지라면 배열(힙을 사용)를 사용했지만, Span<T>의 경우 stackalloc을 안전하기 때문에 힙 사용을 피할 수 있다.  

다음의 수정은 고정 길이 2 문자의 char를 위해 new char[2] 하고 있었던 것을 stackalloc으로 옮겨 놓고 있다.  
```
//before
char[] chars = new char[2] { highSurr, lowSurr };
 
//after
ReadOnlySpan<char> chars = stackalloc char[2] { highSurr, lowSurr };
```
  
그러나 .NET의 구현은 메모리의 스택 영역은 고정 길이 1MB 정도(확실히)이므로 너무 큰 데이터를 스택에 두려고 하면 쉽게 stack overflow를 일으키기도 한다. 방금 같은 고정 길이로 짧은 데이터는 좋지만, 가변 길이의 경우는 조금 궁리해야 한다.  

구체적으로는, 이를테면 "일정 크기 이하의 때만 stackalloc 사용"이라는 분기를 끼운다.  

다음과 같은 조건 연산자는 상당히 자주 나온다.  
```
Span<byte> datetimeBuffer = ((uint)length <= 16) ? stackalloc byte[16] : new byte[length];
```
  
덧붙여서, 다음과 같은 형태도 있다(지금 internal 이지만). StringBuilder 상당의 처리를 초기 버퍼를 stackalloc, 다음 용량을 늘릴 때 ArrayPool을 사용하는 구현. 또한 "일정 크기 이하의 때만 stackalloc 사용"의 최적화의 일종이다.  
  
  

## 네이티브 메모리를 직접
Span<T>를 사용하면 배열도 stackalloc에서 확보한 스택 영역도, 네이티브 메모리도 공통 처리를 쓸 수 있다.  
그래서 기본 상호 운용성 시에 C# 측에서 임시 배열을 확보하고 네이티브 코드로 포인터를 전달하는 이외에, 네이티브 측에서 포인터를 돌려주고 그것을 C#쪽에서 Span<T>을 통해 처리하는 것도 있다.  

이것도 임시 버퍼의 확보가 불필요하게 되므로 성능 개선을 주도하기도 한다.  

최근 [ML.NET](https://github.com/dotnet/machinelearning) 내에서, [TensorFlor과의 상호 운용성](https://github.com/dotnet/machinelearning/blob/e2e1aa8a2b43aa0b5ea5d8b3851b6a0d175f7916/src/Microsoft.ML.TensorFlow/TensorflowTransform.cs)에서도 네이티브 메모리 읽기 및 쓰기에 Span<T>를 사용하기도 한다.  