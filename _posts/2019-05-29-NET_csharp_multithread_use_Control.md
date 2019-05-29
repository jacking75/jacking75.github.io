---
layout: post
title: C# - 다중 스레드에서 컨트롤을 변경 할 때
published: true
categories: [.NET]
tags: c# .net winform
---
출처: MSDN    
  
```
// 델리게이트를 선언한다.
delegate void SetTextCallback(string text);


// 컨트롤의 접근은 따로 함수를 만들어서 접근하도록 한다.
private void SetText(string text)
{

    // InvokeRequired required compares the thread ID of the
    // calling thread to the thread ID of the creating thread.
    // If these threads are different, it returns true.
    if (this.textBox1.InvokeRequired)
    {
        SetTextCallback d = new SetTextCallback(SetText);
        this.Invoke(d, new object[] { text });
     }
    else 
    {
        this.textBox1.Text = text;
     }
}

//스레드에서 아래 함수를 호출하면 된다.
SetTextCallback(SetText); 
```
  