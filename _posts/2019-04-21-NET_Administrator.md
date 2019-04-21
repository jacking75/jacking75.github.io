---
layout: post
title: 프로그램을 관리자 권한으로 실행하는 코드
published: true
categories: [.NET]
tags: .net c# admin
---
[출처](https://qiita.com/tada0724/items/645616a5d1deb55bfdb8 )  
  
권한을 확인하고 관리자 권한이 아니면 다른 프로세스로 관리 권한을 부여하고 다시 시작한다  
```
Thread.GetDomain().SetPrincipalPolicy(PrincipalPolicy.WindowsPrincipal);
var pri = (WindowsPrincipal)Thread.CurrentPrincipal;

//관리자 권한 이외로 실행했다면 다른 프로세스로 이 프로세스를 실행한다
if (!pri.IsInRole(WindowsBuiltInRole.Administrator))
{
    var proc = new ProcessStartInfo()
    {
        WorkingDirectory = Environment.CurrentDirectory,
        FileName = Assembly.GetEntryAssembly().Location,
        Verb = "RunAs"
    };

    if (args.Length >= 1)
        proc.Arguments = string.Join(" ", args);

    //다른 프로세스로 이 프로세스를 실행한다
    Process.Start(proc);

    //현재 프로세스 종료
    return;

}
```
  
위 처리를 Release 빌드 시에만 활성화하여 Visual Studio에서 디버깅 시는 기존 프로세스 그대로 동작하게 하면 디버깅 할 수 있다.   
다음 전체 소스이다.  
```
using System;
using System.Diagnostics;
using System.Reflection;
using System.Security.Principal;
using System.Threading;

class Program
{
    static void Main(string[] args)
    {

#if (DEBUG)
        Thread.GetDomain().SetPrincipalPolicy(PrincipalPolicy.WindowsPrincipal);
        var pri = (WindowsPrincipal)Thread.CurrentPrincipal;

        //관리자 권한 이외로 실행했다면 다른 프로세스로 이 프로세스를 실행한다
        if (!pri.IsInRole(WindowsBuiltInRole.Administrator))
        {
            var proc = new ProcessStartInfo()
            {
                WorkingDirectory = Environment.CurrentDirectory,
                FileName = Assembly.GetEntryAssembly().Location,
                Verb = "RunAs"
            };

            if (args.Length >= 1)
                proc.Arguments = string.Join(" ", args);

            //다른 프로세스로 이 프로세스를 실행한다
            Process.Start(proc);

            //현재 프로세스 종료
            return;

        }
#endif

        /* 주 처리 */

    }
}
```
  
시작할 때 관리자 권한이 있는지 확인하고, 재 시작해서 매니페스트 파일을 사용하지 않고 응용 프로그램을 관리자 권한으로 실행 할 수 있다.