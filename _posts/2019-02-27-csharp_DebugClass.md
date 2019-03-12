---
layout: post
title: C# - Debug 클래스
published: true
categories: [.NET]
tags: c# .net Debug
---
## Debug 클래스
- Debug 클래스는 그 이름대로 디버그 정보를 출력하는 클래스이다.
- Debug 클래스는 System.Diagnostics 이름 공간에 있다.
- 사용 예
  
```
using System.Diagnostics;


Debug.WriteLine("Start Method");
Debug.IndentSize = 4;
Debug.Indent();
Debug.Write("Debug.Write");
Debug.WriteLine("는 개행합니다.");

for (int i = 0; i < 4; i++) {
 Debug.WriteIf(i%2==1,i);
 Debug.WriteLineIf(i % 2 == 1, " 는 기수 입니다.");
}

Debug.WriteLine(e);
Debug.Unindent();
Debug.WriteLine("End Method");
Debug.Flush();
```
  
- 위 코드의 디버그 메시지는 아래와 같다. VS IDE에 출력된다
  
<pre>
Start Method
    Debug.Write는 개행합니다.
    1 는 기수 입니다.
    3 는 기수 입니다.
    System.Windows.Forms.MouseEventArgs
End Method
</pre>
  
  
  
## Debug 클래스 메소드
- IndentSize 프로퍼티
    - 하나의 인덴트에 포함되는 공백의 수를 얻거나 수정
- Indent 메소드
　현재 IndentLevel을 1만큼 증가 시킨다. 
- Unindent 메소드
    - 현재의 IndentLevel을 1만큼 감소 시킨다. 
- Write
- WriteLine
- WriteIf, WriteLineIf 메소드
    - 첫번째 인수 값이 true 때만 디버그 정보를 출력한다
- Flush 메소드
- Assert 메소드
    - Debug.Assert(obj != null, "obj is null");
