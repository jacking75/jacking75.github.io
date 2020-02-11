---
layout: post
title: C# - .NET Core의 WinForms/WPF로 만든 프로그램의 실행 파일 패스
published: true
categories: [.NET]
tags: c# .net winform wpf
---
닷넷프레임워크의 WinForm/WPF는 그 자체로 바로 실행되지만 .NET Core로 만든 것은 `dotnet` 명령어를 사용하여 프로그램을 실행하기 때문에 프로그램의 실행 파일 패스를 얻을 때 이전(닷넷프레임워크)과 다르다.  
`Process.GetCurrentProcess`를 사용해야 한다.  
  
```
using System.Diagnostics;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();

            MessageBox.Show(Process.GetCurrentProcess().MainModule.FileName, "WinForms", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```