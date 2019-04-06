---
layout: post
title: C# - Process
published: true
categories: [.NET]
tags: c# .net process
---
## 실행중인 프로세스트의 디렉토리 위치
  
```
string 현재위치 = Environment.CurrentDirectory;
```
  
  
  
## 다른 프로세스 실행
  
```
System.Diagnostics.Process.Start("실행파일경로\실행파일명.exe",파라메터)
System.Diagnostics.Process.Start("cmd.exe 명령어");
```
  
- C# - 배치 파일 실행하고 출력 결과를 얻는 방법
    - http://www.sysnet.pe.kr/Default.aspx?mode=2&sub=0&detail=1&pageno=0&wid=1810&rssMode=1&wtype=0 
  
  
  
## 프로세스 종료
  
```
System.Diagnostics.Close();                    // 프로세스의 리소스를 해재(종료) 시킨다.
System.Diagnostics.CloseMainWindow();  // UI가 있는 프로세스에 메시지를 보내 종료 시킨다.
System.Diagnostics.Kill();                        // 즉시 프로세스를 종료시킨다.
```
  
  
  
## 프로세스 실행 후 종료까지 대기
  
```
System.Diagnostics.Process p = System.Diagnostics.Process.Start("C:\\test.txt");
p.WaitForExit(); //혹은 시간으로 설정 가능  p.WaitForExit(10000);
```
  
  
  
## 프로세스 실행 후 종료를 비동기로 대기
  
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
  
  
  
## 실행중인 프로세스 찾기
  
```
var exeName = "cmd";
Process[] localByName = Process.GetProcessesByName("LoginServerr");
if (localByName.Length > 0)
{
    return true;
}
```
  
  
  
## 프로세스 타이틀 이름으로 프로세스 찾기
  
```
var allprocess = Process.GetProcesses();
if (allprocess.Count() > 0)
{
    foreach (var pro in allprocess)
    {
        if (pro.MainWindowTitle == "명령 프롬프트")
        {
            Console.WriteLine("{0} 타이틀 프로세스를 찾았음", "명령 프롬프트");
        }
    }
}
```
  
  
  
## Win32 API의 SendMessage 사용
  
```
const int WM_COPYDATA = 0x4A;
[DllImport("user32.dll", CharSet = CharSet.Auto)]
public static extern IntPtr SendMessage(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam);

Process[] AppServerProcess = Process.GetProcessesByName("LoginServer");
//.................
SendMessage(AppServerProcess[0].MainWindowHandle, WM_COPYDATA, IntPtr.Zero, iPtrForCdsMsg);
```
  
  
  
## cpu, 메모리 체크
- wmi 사용 http://www.codeproject.com/csharp/wmi.asp
- 참고: http://code-life.net/?p=1846 , http://dobon.net/vb/dotnet/system/readperformancecounter.html
  
```
// 컴퓨터 전체의 cpu 사용률, 처음에는 0으로 나옴
PerformanceCounter objCPU = new PerformanceCounter("Processor", "% Processor Time","_Total");
label1.text = objCPU.NextValue();

// 특정 프로세스(test.exe)의 CPU 사용률
PerformanceCounter objCPU = new PerformanceCounter("Process", "% Processor Time", "test");
label1.text = objCPU.NextValue();

// 특정 프로세스(test.exe)의 프라이베트 메모리 사용 크기
PerformanceCounter objmemory = new PerformanceCounter("Process", "Working Set - Private", "test");
label1.text = objmemory.NextValue();
```
  