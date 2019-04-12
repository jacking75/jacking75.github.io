---
layout: post
title: C# - ScriptCs
published: true
categories: [.NET]
tags: c# .net ScriptCs
---
## 개요
- 텍스트 파일에 적은 C# 코드를 컴파일러를 사용하지 않고 명령어로 실행할 수 있는 툴이다.
- Roslyn을 힘을 이용한 것이다.
- .NET Framework 4.5 이상이 필요하다.
- 공식 사이트   http://scriptcs.net/
- 샘플 사이트   https://github.com/scriptcs/scriptcs-samples
- 위키         https://github.com/scriptcs/scriptcs/wiki
- 설치
    - Windows의 패키지 관리 소프트인 chocolatey( http://chocolatey.org/ )을 설치한다.
    - 다음의 명령어를 실행한다(아마 콘솔창을 관리자 권한으로 실행해야 할 것이다).
        - cinst scriptcs
        - %APPDATA%\scriptcs\ 에 설치 되어 잇다.
        - 설치 후 scriptcs를 업그레이드 하고 싶다면 cup scriptcs
- 설치가 끝나면 자동으로 path가 설정되어서 어디에서나 실행할 수 있다.
- dynamic 타입은 사용할 수 없다.
- Atom 연동
    - https://github.com/scriptcs/scriptcs/wiki/Running-scripts-in-Atom
    - 설치 후 스크립트 후 Alt+R 로 실행할 수 있다.
- Visual Studio에서 디버깅 하기
    - https://github.com/scriptcs/scriptcs/wiki/Debugging-a-script
- 소개 문서
    - http://www.slideshare.net/DavidLeePerish/script-cs-for-business-and-pleasure
    - http://www.slideshare.net/FilipW/introduction-to-scriptcs
  
  
  
## REPL
- scriptcs.exe를 실행하면 REPL을 동작 시킬 수 있다.
  
```
C:\> scriptcs
scriptcs (ctrl-c or blank to exit)

> var message = "Hello, world!";
> Console.WriteLine(message);
Hello, world!
>
```
  
- using, namespace, class, Main 메소드도 필요없다.
- REPL뿐만이 아닌 파일에 적힌 코드를 실행할 수도 있다.
  
  
  
## 스크립트 파일 사용하기
- 인수에 파일 이름을 넘겨 주면 실행한다. 확장자는 csx을 추천. csx가 아니어도 괜찮다.
- 문자 코드는 BOM 있는 UTF-8로 해야한다. Roslyn는 전달 된 문자열을 무조건 BOM 붙은 UTF-8로 해석한다.
- 샘플을 test.csx으로 저장 "scriptcs test.csx"라고 실행하면 콘솔에 "Hello, world!"가 나타난다.
  
  
  
## 다른 스크립트 파일을 읽기
Other.csx  
```
public class Hoge {
    public string Fuga {get; set;}
    public int Piyo {get; set;}

    public string Foo() {
        return "Foo";
    }
}
```
  
"#load"를 사용하여 스크립트 내에서 읽어들일 수 있다. REPL이나 스크립트 파일 모두에서 사용할 수 있다.
  
```
C:\> scriptcs
scriptcs (ctrl-c or blank to exit)

> #load "Hoge.csx";
> var hoge = new Hoge { Fuga = "fuga", Piyo = 1 };
> hoge
{
  "$id": "1",
  "Fuga": "fuga",
  "Piyo": 1
}
> hoge.Foo();
"Foo"
>
```
  
  
  
## 스크립트에서 어셈블리 로딩하기
- "#load"와 같은 방법으로 #r을 사용하면 dll을 로드 할 수 있다. GitHub의 샘플을 참조해라.
- 이 방법을 사용해서 자신이 만든 dll도 호출해서 사용할 수 있다.
- 상대 경로를 지정해서 dll을 로드해야 한다.
- 아래의 어셈블리는 자동으로 로드한다.
    - System
    - System.Core
    - System.Data
    - System.Data.DataSetExtensions
    - System.Xml
    - System.Xml.Linq
- 또 아래의 namespace는 암묵적으로 using이 지정되어 있다.
    - System
    - System.Collections.Generic
    - System.Linq
    - System.Text
    - System.Threading.Tasks
- "일반적으로 콘솔 응용 프로그램을 만들 때의 설정과 같다"라고 기억해두면 좋다.
  
  
  
## Nuget 사용하기
- Nuget을 내장하고 있다. 설치 방법도 매우 간단하다.
- 예를 들어 Json.NET의 경우
  
```
scriptcs -install Newtonsoft.Json
```
  
- 당연히 종속성이 있으면 그것도 함께 설치한다.
- 설치 한 패키지는 "scriptcs_packages"란 디렉토리에 있다. 여기에 저장된 것은 #r도 #load도 필요 없다.
- 사용하고 싶을 때 using에서 namespace를 지정하면 OK.
- 주의해야 할 것이 있다. scriptcs_packages도 scriptcs.exe를 호출 한 디렉토리(현재 디렉토리)에 생성/저장하고 스크립트 내에서도 현재 디렉토리에 scriptcs_packages가 없으면 패키지를 로드하지 않는다.
  
  
  
## Script Packs
- ScriptCS에서 사용하는 것을 목적으로 하는 패키지 군이다.
- 대부분 Nuget로 배포하기 때문에 역시 쉽게 설치할 수 있습니다.
- 이들은 ScriptCs.Contracts을 계승하고 Require<TScriptPack>()라는 형식으로 호출 할 수 있다.
- 예를 GitHub의 예를 봐라.  https://github.com/scriptcs/scriptcs#bootstrap-scripts-with-script-packs
  
  
### Script Packs 소개
- ScriptCs.Rebus
    - 스크립트 간에 메시지 전달.
    - IPC 이외에 RabbitMQ, Azure Service Bus 사용 가능
    - https://github.com/scriptcs-contrib/scriptcs-rebus
- ScriptCs.SsRedis
    - Redis 사용
    - https://github.com/filipw/ScriptCs.SsRedis
- ScriptCs.SignalR
    - 서버 역할을 한다.
    - https://github.com/MartinDoms/ScriptCs.SignalR
- Shovel
    - 빌드 시스템 만들기. 빌드 후 복사 등을 다른 일을 할 수 있다.
    - https://github.com/jancowol/Shovel
- ScriptCs.Wpf
    - WPF UI를 사용할 수 있다. 코드를 직접 기술하거나 xaml을 사용.
    - https://github.com/scriptcs-contrib/scriptcs-wpf
- ScriptCs.WebApi
    - https://github.com/scriptcs-contrib/scriptcs-webapi
- ScriptCs.SimpleWeb
    - https://github.com/ianbattersby/scriptcs-simpleweb
- ScriptCs.Request
    - http 클라이언트.
    - https://github.com/martinobrink/ScriptCs.Request
- scriptcs-objectdumper
    - 오브젝트 내용을 보여준다. LinqPad의 Dump()와 비슷
    - https://github.com/paulbouwer/scriptcs-objectdumper
- ScriptCs.Nest
    - Elasticsearch의 닷넷용 클라이언트 라이브러리 Nest를 사용.
    - https://github.com/Mpdreamz/scriptcs-nest
- ScriptCs.Net
    - TCP 서버/클라이언트
    - https://github.com/scriptcs-contrib/scriptcs-net
- ScriptCs.Nancy
    - https://github.com/adamralph/scriptcs-nancy
- scriptcs-logger
    - https://github.com/paulbouwer/scriptcs-logger
- ScriptCs.Gui
    - 간단하게 gui를 사용할 수 있다.
    - https://github.com/hemme/scriptcs-gui
- scriptcs-edge
    - Node.js에서 닷넷을 사용할 수 있는 edge를 사용할 수 있다.
    - https://github.com/scriptcs-contrib/scriptcs-edge
- Bau
    - task runner
    - https://github.com/bau-build/bau
- ScriptCs.AzureMobileServices, ScriptCs.AzureMediaServices, ScriptCs.AzureManagement
  
  
  
## 실행 시 인수 전달
- ScriptCS도 시작할 때 인수를 설정할 수 있다.
- Env.ScriptArgs에 ReadOnlyCollection<string>으로 들어 있다. ReadOnlyCollection는 IEnumerable을 구현하지 않기 때문에 주의.
  
```
Console.WriteLine (Env.ScriptArgs [0]);
```
  
- 인수를 전달
  
```
C : \> scriptcs test.csx - test
```
  
- 멀티 바이트 문자와 공백을 포함 하는 인수는 큰 따옴표로 사용하면 OK.
  
```
  C : \> scriptcs test.csx - "시샵" "부호가 부호가"
```
  
- REPL에서는 시작할 때 인수를 전달할 수 없다.
  
  
  
## 참고
- http://outofmem.hatenablog.com/entry/2015/02/04/014352  
