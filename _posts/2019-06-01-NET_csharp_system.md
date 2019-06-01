---
layout: post
title: C# - 시스템 프로그래밍
published: true
categories: [.NET]
tags: c# .net
---
### 다른 프로세스 실행
  
```
System.Diagnostics.Process.Start("실행파일경로\실행파일명.exe",파라메터)
System.Diagnostics.Process.Start("cmd.exe 명령어");
```
  
  
  
### 프로세스 종료
```
System.Diagnostics.Close();                    // 프로세스의 리소스를 해재(종료) 시킨다.
System.Diagnostics.CloseMainWindow();  // UI가 있는 프로세스에 메시지를 보내 종료 시킨다.
System.Diagnostics.Kill();                        // 즉시 프로세스를 종료시킨다.
```
  
  
  
### 프로세스 실행 후 종료까지 대기
  
```
System.Diagnostics.Process p = System.Diagnostics.Process.Start("C:\\test.txt");
p.WaitForExit(); //혹은 시간으로 설정 가능  p.WaitForExit(10000);
```
  
  
  
### 프로세스 실행 후 종료를 비동기로 대기
  
```
private void button1_Click(object sender, System.EventArgs e)
{
    System.Diagnostics.Process p = new System.Diagnostics.Process();
    p.StartInfo.FileName = "notepad.exe";
    p.SynchronizingObject = this;
    
    p.Exited += new EventHandler(p_Exited);
    p.EnableRaisingEvents = true;
    
    p.Start();
}

private void p_Exited(object sender, EventArgs e)
{
    // 프로세스가 종료하면 이벤트 실행
}
```
  
  
  
### 실행중인 프로세스 찾기
  
```
System.Diagnostics.Process.GetProcessByName( "실행파일 이름" );

Process[] localByName = Process.GetProcessesByName("LoginServerr");
if (localByName.Length > 0)
{
     return true;
}
```
  
  
  
### Win32 API의 SendMessage 사용
  
```
const int WM_COPYDATA = 0x4A;
[DllImport("user32.dll", CharSet = CharSet.Auto)]
public static extern IntPtr SendMessage(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam);
 
Process[] AppServerProcess = Process.GetProcessesByName("LoginServer");
//.................
SendMessage(AppServerProcess[0].MainWindowHandle, WM_COPYDATA, IntPtr.Zero, iPtrForCdsMsg);
```
  
  
  
### cpu, 메모리 체크
  
```
// cpu 사용률, 처음에는 0으로 나옴
PerformanceCounter objCPU = new PerformanceCounter("Processor", "% Processor Time","_Total");
label1.text = objCPU.NextValue();

// wmi 사용
http://www.codeproject.com/csharp/wmi.asp


//메모리 총사용량
Process[] allPro = Process.GetProcesses();
long memory = 0;
foreach (Process pro in allPro)
{
    memory += pro.VirtualMemorySize64;
}  
```
  