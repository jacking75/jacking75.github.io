---
layout: post
title: C# - WPF - 멀티 스레드에서 컨트롤 조작
published: true
categories: [.NET]
tags: c# .net wpf
---
from : MSDN  
    
```
// 아래의 예는 리스트박스에 새로운 데이터를 추가하는 것이다.
private delegate void ListBoxDelegate(string arg);

void SetStateText(string state)
{
     this.listBox1.Dispatcher.BeginInvoke(
                System.Windows.Threading.DispatcherPriority.Normal,
                new ListBoxDelegate(UpdateListBox), state);
}

private void UpdateListBox(string state)
{
     this.listBox1.Items.Add(state);
}


// 메인 스레드가 아닌 곳에서는 SetStateText(string state)을 호출하여 리스트박스를 변경한다. 
```
  