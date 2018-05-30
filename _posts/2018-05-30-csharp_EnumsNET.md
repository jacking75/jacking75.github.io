---
layout: post
title: .NET - Enums.NET
published: true
categories: [.NET]
tags: c# .net enum
---
[Enums.NET](https://github.com/TylerBrinkley/Enums.NET)  

C#의 enum을 직접 사용하는 것에 비해 Enums.NET을 사용하는 것이 더 빠름.  
아래는 비교 코드.  
[출처](http://baba-s.hatenablog.com/entry/2018/03/26/102000)    
  
```
using EnumsNET;
using System;
using System.Diagnostics;
using System.Text;

public static class Program
{
    private enum TYPE
    {
        GRASS,
        FIRE,
        WATER,
    }

    private static void Main()
    {
        int count = 10000000;

        var sb = new StringBuilder();
        var sw = new Stopwatch();

        // 기본 enum
        sw.Start();
        for ( int i = 0; i < count; i++ )
        {
            var result = Enum.GetValues( typeof( TYPE ) ) as TYPE[];
        }
        sw.Stop();

        sb.AppendFormat( "Enum: {0}", sw.Elapsed ).AppendLine();

        // Enums.NET
        sw.Restart();
        for ( int i = 0; i < count; i++ )
        {
            var result = Enums.GetMembers<TYPE>();
        }
        sw.Stop();

        sb.AppendFormat( "Enums: {0}", sw.Elapsed ).AppendLine();

        Console.WriteLine( sb );
    }
}
```   
결과 기본 enum은 9.61초, Enums.NET은 0.42초  
  
기능   
```
using EnumsNET;
using System;

public static class Program
{
    [Flags]
    private enum TYPE
    {
        NONE = 0,
        GRASS = 1,
        FIRE = 2,
        WATER = 4,
    }

    private static void Main()
    {
        // 열거자의 모든 요소 얻기
        var members = Enums.GetMembers<TYPE>();
        var values = Enums.GetValues<TYPE>( EnumMemberSelection.Distinct );

        // 열거자의 요수 수 얻기
        Console.WriteLine( Enums.GetMemberCount<TYPE>() ); // 4

        // 지정된 모든 플래그를 포함한 경우 true
        Console.WriteLine( TYPE.GRASS.HasAllFlags( TYPE.GRASS ) ); // True

        // 지정된 어떤 플래그를 포함하는 경우 true
        Console.WriteLine( TYPE.GRASS.HasAnyFlags( TYPE.GRASS ) ); // True

        // 지정된 플래그를 삭제하고 반환
        Console.WriteLine( TYPE.GRASS.RemoveFlags( TYPE.GRASS ) ); // NONE

        // ToString
        Console.WriteLine( TYPE.GRASS.AsString() ); // GRASS

        // 요소 이름 얻기
        Console.WriteLine( TYPE.GRASS.GetName() ); // GRASS

        // 지정된 요소를 열거자 타입으로 포함하는 경우 true
        Console.WriteLine( ( ( TYPE )4 ).IsValid() ); // True

        // 지정된 문자열을 열거 타입으로 변환
        Console.WriteLine( Enums.Parse<TYPE>( "GRASS" ) ); // GRASS
    }
}
```  

