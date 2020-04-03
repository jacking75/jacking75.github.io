---
layout: post
title: C# - 단일 실행 파일로 Ubuntu 18.04 에서 실행하기
published: true
categories: [.NET]
tags: .net c# .netcore single
---
[출처](https://qiita.com/kaysquare1231/items/700be91a1f4d410c1d5e )  
  
## 콘솔 앱을 만든다
  
```
Program.cs
using System;
using System.Threading;

namespace SingleFileConsole
{
    class Program
    {
        static void Main(string[] args)
        {
            while (true)
            {
                Console.WriteLine("Hello World!");
                Thread.Sleep(1000);
            }
        }
    }
}
```
  
  
  
## 애플리케이션을 발행한다
리눅스 플랫폼으로 지정해서 싱글 파일로 발행한다.  
  
```
dotnet publish -c Release -r linux-x64 /p:PublishSingleFile=true
```
  
  
## 애플리케이션을 실행한다
Windows 10에서 개발한 애플리케이션을 Ubuntu 18.04 에 복사한다.  
터미널을 실행하고 애플리케이션을 배치한 디렉토리로 이동한다.  
`chmod +x SingleFileConsole`로 애플리케이션을 실행 가능 형식으로 한다. SingleFileConsole는 위에서 만든 실행 파일의 이름이다.  
`./SingleFileConsole` 로 애플리케이션을 실행한다.  