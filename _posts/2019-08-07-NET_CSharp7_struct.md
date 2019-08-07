---
layout: post
title: C# 7.2의 구조체의 성능
published: true
categories: [.NET]
tags: .net c# .netcore struct
---
[출처](https://www.infoq.com/news/2018/07/structs-performance-csharp )  

C# 컴파일러는 readonly를 따르는 일부 조건에서 구조체의 방어적 복사(defensive copy)를 생성한다.  
이 문제는 잘 알려져 있고 문서화 되어 있지만 C# 7.2의 일부 기능과 관련이 있기 때문에 검토 가치가 있다.  
in및 ref readonly 키워드는 문제의 발생을 높이고 readonly 구조체는 수정 방법을 제공한다.  

C# 구조체는 메모리 할당/해제 비용을 피하려고 하고, 일반적으로 고성능을 목적으로 사용된다. 그런데 이것을 제한하는 잠재적인 함정이 있다.  
C# 7.2에서는 잘 발생하는 경우에 대처하기 위해 readonly 구조체 라는 개선 기능이 추가 되어 있다.  

아래의 경우 C# 컴파일러는 구조체의 복사본을 생성한다.  
- struct 시그니쳐가 readonly가 아닌 경우
- 구조체 변수에 readonly 한정자가 있는 경우
- 메소드(속성 포함)가 호출 되는 경우  

```
public struct SomeStruct
{
	private int _x;

	public int X { get { return _x; } }
}

private readonly SomeStruct s = new SomeStruct(42);

s.X; // 컴파일러는 방어적 복사를 만든다.
```  

x가 in- 매개 변수, ref readonly 지역 변수, readonly 참조에 의한 값을 반환하는 메서드 호출의 결과 등의 경우 동일한 규칙이 적용된다.  

```
public void BadFunction(in SomeStruct s)
{
  s.X; // 컴파일러는 방어적 복사를 만든다.
}
```  

C# 7.2에서는 구조체를 readonly 라고 선언 할 수 있는 기능을 추가함으로써 방어 복사를 피하기 위한 해결책을 제공하고 있다.  readonly 라고 선언한 구조체는 속성 세터를 가질 수 없으며 구조체 멤버에 대한 할당이 금지된다.  

방어 복사의 문제는 정적 분석에 의해 감지 할 수 있다. [ErrorProne.NET](https://github.com/SergeyTeplyakov/ErrorProne.NET )는 Java 용 정적 분석 도구 ErrorProne에서 영감을 받은 것이다. .NET 포트는 Roslyn 분석기의 집합체로 되어 있어 정확성과 성능에 중점을 두고있다. 이 일부로서 [구조체에 초점을 맞춘 것이 있고](https://www.nuget.org/packages/ErrorProne.NET.Structs/0.1.0 ) Nuget 패키지로 이용 할 수있다.
