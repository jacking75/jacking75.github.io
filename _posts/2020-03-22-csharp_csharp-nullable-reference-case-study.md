---
layout: post
title: C# - 프로젝트에 C# 8과 null 허용 참조 타입 대응시키기
published: true
categories: [.NET]
tags: c# .net null c#8
---
[출처](https://www.infoq.com/articles/csharp-nullable-reference-case-study )  
  
C# 7의 클래스 라이브러리를 null 허용 참조 타입을 사용하는 C# 8로 업그레이드 하는 사례에 관한 글이다.  
여기에 사용된 [Tortuga Anchor](https://github.com/docevaad/Anchor ) 프로젝트는 MVVM 형식의 기본 클래스와 리플렉션 코드, 다양한 유틸리티 함수를 모아 놓은 것이다. 적당히 작고, 관용적 패턴과 일반적이지 않은 패턴이 혼재하고 있기 때문에이 프로젝트를 선택했다.  
  
  
## 프로젝트 설정
현재 null 허용 참조 타입을 사용 할 수 있는 것은 .NET Standard 프로젝트와 .NET Core 프로젝트 뿐이지만, Visual Studio 2019이 출시 되기 전까지는 .NET Framework에서 지원될 것으로 생각한다.  
  
프로젝트 파일에 아래 설정을 추가하거나 수정해야한다.  
```
</PropertyGroup>
    <LangVersion>8.0</LangVersion>
    <NullableContextOptions>enable</NullableContextOptions>
</PropertyGroup>
```
  
저장하면 즉시 null 허용 오류가 표시 되기 시작 되어야 하지만, 그렇지 않은 경우는 프로젝트를 빌드해 본다.  
  
  
## 타입이 null 허용임을 나타내기
인터페이스 메소드 GetPreviousValue의 반환 값은 null 허용 형식이지만 이것을 명시 적으로 하기 위해 object에 null 허용 참조 타입 수정자(?)를 추가한다.  
  
```
object? GetPreviousValue(string propertyName);
```
  
앞의 컴파일러 오류의 대부분은 변수, 매개 변수, 반환 타입이 타입 한정자를 추가하면 해소될 것이다.  
  
  
  
## 지연 읽기 속성
속성의 연산 처리가 비용이 비싼 경우, 지연 읽기 패턴을 이용하는 것이 많다. 이 패턴은 전용 필드가 null이면 이 값이 미 생성이라는 것을 나타낸다.  
  
C# 8은 이 상황을 잘 처리 해준다. 코드를 수정하지 않고 아래 코드를 제대로 분석하면 반환 값이 null 허용 타입임에도 불구하고 Getter의 결과는 항상 null이 아님을 알 수 있다.  
  
```
string? m_CSharpFullName;
public string CSharpFullName
{
    get
    {
        if (m_CSharpFullName == null)
        {
            var result = new StringBuilder(m_TypeInfo.ToString().Length);
            BuildCSharpFullName(m_TypeInfo.AsType(), null, result);
            m_CSharpFullName = result.ToString();
        }
        return m_CSharpFullName;
    }
}
```
  
이 경우에는 잠재적으로 경쟁 조건이 있는 것에 주목한다. 이론적으로 다른 스레드가 m_CSharpFullName 값을(역주 : if 블록 내에서) null로 리턴 할 수 있지만, 컴파일러는 이것을 찾을 수 없다. 따라서 멀티 스레드 코드를 다루는 경우에는 특히 주의가 필요하다.  
  
  
  
## 변수의 null 허용 성이 다른 변수에 의해 결정된다
다음 코드 예제 클래스는 m_ItemPropertyChanged이 null이 아닌 경우에만 m_ListeningToItemEvents이 사실인 것처럼 디자인 되어 있다.  
이 규칙은 컴파일러는 모른다. 그래서 null 허용 연산자(!)를 추가하여 변수(이 경우 m_ItemPropertyChanged)가 이 시점에서는 null이 되지 않는다고 표시 할 수 있다.  
  
```
if (m_ListeningToItemEvents)
{
    if (item is INotifyPropertyChangedWeak)
        ((INotifyPropertyChangedWeak)item).AddHandler(m_ItemPropertyChanged!);
    else if (item is INotifyPropertyChanged)
        ((INotifyPropertyChanged)item).PropertyChanged += OnItemPropertyChanged;
}
```
  
  
  
## 명시적 캐스트에 의한 오판정 수정
아래 예제에서는 컴파일러가 m_Base. Values의 null 허용 성이 IEnumerable<TValue>와 호환성이 없다는 잘못된 보고를 한다. 경고를 없애려면 아래와 같이 명시적 캐스트를 추가했다.  
  
```
readonly Dictionary<ValueTuple<TKey1, TKey2>, TValue> m_Base;
IEnumerable<TValue> IReadOnlyDictionary<ValueTuple<TKey1, TKey2>, TValue>.Values
{
    get { return (IEnumerable<TValue>)m_Base.Values; }
}
```
  
그럼 이번에는 컴파일러가 이 행에 중복 캐스트가 플래그를 세운다. 보통 이것은 컴파일러의 메시지이며, 경고는 아니지만, 출시 시에는 수정될 예정이다.
  
  
  
## 임시 변수 또는 조건부 캐스트에 의한 오판종 수정
다음 예제에서는 CancelEdit 행에 오류가 있음을 표시한다. 직전의 if 문에서 item.Valuenull가 아닌 것이 보증 되어 있어도 컴파일러는 다음 item.Value을 읽을 때도 null이 없다고는 믿지 않는다.  
  
```
foreach (var item in m_CheckpointValues)
{
    if (item.Value is IEditableObject)
        ((IEditableObject)item.Value).CancelEdit();
}
```
  
컴파일러를 납득시키는 방법 중 하나는 item.Value을 임시 변수에 저장하는 것이다.  
```
foreach (var item in m_CheckpointValues)
{
    object? value = item.Value;
    if (value is IEditableObject)
        ((IEditableObject)value).CancelEdit();
}
```
  
그러나 이 경우에는 조건부 캐스트( "as"연산자)를 사용하여 조건부 메서드 호출( "?" 연산자)를 계속하는 것으로 더 간단하게 할 수 있다.  
```
foreach (var item in m_CheckpointValues)
{
    (item.Value as IEditableObject)?.CancelEdit();
}
```
  
  
  
## 범용 타입과 null을 허용 타입
범용 타입을 많이 사용하면, null 허용 유형의 문제에 부딪 힐 수 있다. 아래 대리자를 생각해 보자.  
  
```
public delegate void ValueChanged<in T>(T oldValue, T newValue);
```
  
이 대리자의 설계 의도는 oldValue과 newValue는 함께 null을 허용한다. 그렇다면 물음표를 2 개 붙이면 좋다,라고 생각할지도 모른다. 그런데 그렇게 하면 아래와 같은 오류 메시지가 반환된다.  
> Error CS8627 A nullable type parameter must be known to be a value type or non-nullable reference type. Consider adding a 'class', 'struct', or type constraint.
  
값과 참조 형식을 모두 지원해야 하는 경우 이를 쉽게 해결할 방법은 없다. 형식 제약 조건은 "or"를 설명 할 수 없기 때문에, 클래스와 구조체 각각 대리자를 준비해야한다.  
```
public delegate void ValueChanged<in T>(T? oldValue, T? newValue) where T : class;
public delegate void ValueChanged<T>(T? oldValue, T? newValue) where T : struct;
```
  
그러나이 방법으로는 잘 되지 않는다. 두 대리자가 같은 이름이기 때문이다. 다른 이름을 붙이면 좋겠지만, 그러면 사용하는 코드도 모두 2개로 해야한다.  
  
다행히도, C# 이스케이프 값이 있다. #nullable 지시어를 사용하면 C# 7의 의미를 되 돌리는 것이 가능하기 때문에 코드는 계속 의도대로 작동한다.  
```
#nullable disable
public delegate void ValueChanged<in T>(T oldValue, T newValue);
#nullable enable
```
  
이 해결 방법은 문제가 없는 이유는 없다. null 허용 참조 기능의 비활성화는 전부 혹은 모두 아닌 것이므로 oldValue와 newValue을 null 허용 할 수 없다.  
  
  
  
## 생성자, 디시리얼라이저 및 초기화 메소드
다음의 예에서는 시리얼 라이저가 사용하는 몇 가지 기술에 대해 알고 있어야한다. 별로 알려져 있지 않지만, 클래스의 생성자를 우회하는 FormatterServices.GetUninitializedObject. 라고 하는 기능이 있다. DataContractSerializer 등 일부 시리얼라이저는 이것을 사용하여 성능을 향상하고 있다.  
  
생성자의 로직을 반드시 실행해야 하는 경우는 어떻게 될까? OnDeserializing 속성은 이것을 위해 있다. 이 속성은 대리 생성자 역할을 하고 GetUninitializedObject 후에 호출된다.  
  
중복 및 오류의 가능성을 줄이기 위해 아래의 예와 같은 일반적인 초기화 메소드가 사용되는 경우가 많다.  
```
protected AbstractModelBase()
{
    Initialize();
}
 [OnDeserializing]
void _ModelBase_OnDeserializing(StreamingContext context)
{
    Initialize();
}
void Initialize()
{
    m_PropertyChangedEventManager = new PropertyChangedEventManager(this);
    m_Errors = new ErrorsDictionary();
}
```
  
이것은 null 검사기에게 문제가 된다. 조금 전의 두 변수가 생성자에서 명시적으로 설정되어 있지 않기 때문에, 초기화 되지 않은 것으로 플래그 지정되는 것이다. 즉, 오류를 방지하기 위해 잘라 내기 작업을 할 필요가 있다는 것이다.  
  
OnDeserializing 메소드를 포함하는 것을 잊는 경우 위험이 있다. null 검사기는 OnDeserializing 메소드를 이해하지 않기 때문에 의도하지 않은 null의 가능성에 대해 경고 할 수 없다.  
  
대부분의 개발자는 이 행동을 이해하기 어렵다고 생각한다. 따라서 .NET Core에서는 DataContractSerializer가 생성자를 호출하게 될 예정이다.   
그러나 .NET Standard를 대상으로 하는 경우에는이 행동의 차이를 고려하여 직렬화 코드를 .NET Framework 및 .NET Core 모두에서 테스트 해야한다.  
  
  
  
## NULL 허용 매개 변수와 CallerMemberName
이 라이브러리는 CallerMemberName 패턴이 많이 사용되고 있다. 사용하는 특성을 따서 명명 된 이 패턴의 기본적인 아이디어는 메소드의 마지막에 선택 매개 변수를 추가한다는 것이다. 컴파일러는이 CallerMemberName를 보고, 이 파라미터의 값을 암시적으로 제공한다.  
  
```
public override bool IsDefined([CallerMemberName] string propertyName = null)
```
  
propertyNameparameter을 명시적으로 null 설정하는 것도 이론적으로는 가능하지만, 그렇게해서는 안 된다는 것은 널리 알려져 있다. 예기치 않은 오류가 발생할 수 있기 때문이다.  
  
이 코드를 C# 8로 변환 할 때 매개 변수를 nulll nullable 형식으로 표시하고 싶어 질지도 모르만, 메소드는 실제로는 null를 처리하도록 설계되어 있지 않기 때문에 이것은 오해를 초래 할 수 있다. 그렇지 않고 null을 빈 문자열로 대체 할 것이다.  
```
public override bool IsDefined([CallerMemberName] string propertyName = "")
```
  
  
  
## 그래도 null 인수 검사가 필요한가?
광범위하게 사용되는(NGet 등) 라이브러리를 개발하고 있다면, Yes 이다. 모든 공개 메소드는 여전히 null 인수 검사를 수행해야한다.  
라이브러리를 사용하는 애플리케이션이 null 허용 참조 타입을 사용하는 것은 없다. 실제 C# 8을 전혀 사용하지 않을지도 모른다.  
  
모든 응용 프로그램 코드에서 null 허용 참조 타입이 사용 되고 있다 해도 답은 역시 "어쩌면 YES"이다. 이론 상으로는 예기치 않은 null를 받지 않을 것이지만, 동적 코드와 리플렉션 또는 null 허용 연산자 (!)의 오용에 의해 섞여 올 가능성이 있기 때문이다.  
  
  
  
## 결론
60 미만의 클래스 파일의 프로젝트로, 이 중 24 파일에 변경이 필요했다.  
하지만 특별히 중요한 것은 아니라 전체 프로세스에 필요한 것은 1 시간 미만이었다.  
결과적으로 작업은 그리 어려운 것이 아니라 대체로 예상대로 작동한다.  
이 기능은 대부분의 프로젝트에 메리트가 있다고 생각되기 때문에 C# 8이 출시 된다면 적극적으로 이용해야한다.  
  