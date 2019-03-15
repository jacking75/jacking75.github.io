---
layout: post
title: C# - Enum
published: true
categories: [.NET]
tags: c# .net enum
---
## Enum의 요소를 foreach로 열거하기
  
```
void Main()
{
    foreach (TEST val in Enum.GetValues(typeof(TEST)))
    {
        Console.WriteLine("{0} - {1}", val, (int)val);
    }
}

enum TEST
{
    ITEM_1 = 1,
    ITEM_2 = 10,
    ITEM_3 = 21,
}
```
  
<pre>
결과
ITEM_1 - 1
ITEM_2 - 10
ITEM_3 - 21
</pre>
  
  
  
## enum의 개수 얻기, enum 요소에 없는 값을 enum으로 변환 시
  
```
void Main()
{
    var enumCount = Enum.GetNames(typeof(TEST)).Length;
    Console.WriteLine("TEST enum 개수: {0}", enumCount);

    Console.WriteLine("TEST enum 1: {0}", (TEST)1);
    Console.WriteLine("TEST enum 2: {0}", (TEST)2);
    Console.WriteLine("TEST enum 3: {0}", (TEST)21);
}

enum TEST
{
    ITEM_1 = 1,
    ITEM_2 = 10,
    ITEM_3 = 21,
}
```
  
<pre>
결과
TEST enum 개수: 3
TEST enum 1: ITEM_1
TEST enum 2: 2
TEST enum 3: ITEM_3
</pre>
  
  
  
## 정수 값이 enum에 정의 되어 있는지 조사
  
```
void Main()
{
    int n = 10;

    if (Enum.IsDefined(typeof(TEST_ENUM), n))
        Console.WriteLine("정수 값이 있다");
    else
        Console.WriteLine("정수 값이 없다");

    n = 11;
    if (Enum.IsDefined(typeof(TEST_ENUM), n))
        Console.WriteLine("정수 값이 있다");
    else
        Console.WriteLine("정수 값이 없다");
}

enum TEST_ENUM
{
    ITEM_1 = 1,
    ITEM_2 = 10,
    ITEM_3 = 21,
}
```
  
<pre>
결과
정수 값이 있다
정수 값이 없다
</pre>
  
  
  
## Flag 판정
  
```
[Flags]
public enum BorderSides { None=0, Left=1, Right=2, Top=4, Bottom=8 }

void Main()
{
    var borderSides = BorderSides.Left | BorderSides.Right;
    borderSides.HasFlag(BorderSides.Left).Dump();

    ((borderSides & BorderSides.Right) == BorderSides.Right).Dump();
}
```
  
<pre>
결과
True
True
</pre>
  