---
layout: post
title: C# - 콘솔 키보드 입력, 프로그램 종료 키 조사
published: true
categories: [.NET]
tags: c# .net 키보드 keyboard 콘솔 console
---
## 키보드 입력
  
```
class Program
{
    static void Main(string[] args)
    {
        System.Threading.ThreadPool.QueueUserWorkItem(new System.Threading.WaitCallback(키보드입력조사), null);

        while (true)
        {
            ......
            System.Threading.Thread.Sleep(128);
        }
    }

    static void 키보드입력조사(object userState)
    {
        while (true)
        {
            var command = Console.ReadLine();

            if (command == "종료")
            {
                .............
            }

        }
    }
}
```
  
  
  
## 프로그램 종료 키 조사
  
```
Console.CancelKeyPress += new ConsoleCancelEventHandler(CancelHandler);

static void CancelHandler(object sender, ConsoleCancelEventArgs args)
{
    if (args.SpecialKey == ConsoleSpecialKey.ControlC)
    {
        Console.WriteLine("Ctrl+C 키를 눌렀으므로 종료 처리를 한다.");

        ServerLogic.ServerInit.Destory();

        IsWating = false;
    }

}
```
  
