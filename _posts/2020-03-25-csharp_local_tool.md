---
layout: post
title: C# - .NET 로컬 툴 사용법
published: true
categories: [.NET]
tags: .net c# localTool tool
---
[출처](https://qiita.com/nogic1008/items/ce2dec260aa31bf205d8 )  
  
## .NET Core도구?
[.NET core 글로벌 툴](https://docs.microsoft.com/ko-kr/dotnet/core/tools/global-tools)  
  
.NET Core 2.1에서 NuGet에 게재된 콘솔 응용 프로그램을 CLI 상에서 인스톨/실행/업데이트/언인스톨 하는 구조가 생겼다.  
이들의 애플리케이션을 총칭해서 ".NET Core 툴" 이라고 부른다.  
npm에서 말하는 npm install -g tool 같은 것이다.  
  
장점은 아래와 같다.
- dotnet 명령어를 사용할 수 있다 = .NET Core SDK가 들어가 있으므로, 거의 확실하게 사전 준비 없이 툴을 사용할 수 있다
- (스크립트 명령과 비교해서)플랫폼의 차이를. NET Core가 흡수한다
- (기존 NuGet에서의 인스톨과 비교해서)PATH가 통하므로 실행 패스가 번잡해지지 않는다
    - Nuget: ~/.nuget/packages/패키지 이름/버전/...
    - Tool: ~/.dotnet/tools
- (수동으로 툴 배포 보다)NuGet 구조를 그대로 이용할 수 있다
    - 파일 서버에 놓아 두는 것만으로 프라이빗 레파지토리로서 전송할 수 있다
  
  
  
## .NET core글로벌 툴의 사용 사례
  
### 설치 
`dotnet tool install -g` 툴의 패키지 이름  
  
  
### 실행 
인스톨 하는 곳은. NET Core SDK에 의한 PATH가 통하므로, 그대로 실행 가능  
도구의 실행 파일 이름 arg1 arg2  
  
dotnet [명령어 이름]으로 부르게 만들어져 있는 경우  
`dotnet 명령 이름 arg1 arg2`  
  
업데이트   
`dotnet tool update -g` 툴의 패키지 이름   
  
언인스톨   
`dotnet tool uninstall -g` 툴의 패키지 이름   
   
  
  
## .NET Core 로컬 툴?
npm의 글로벌 인스톨이 안고 있는 것처럼. NET Core의 글로벌 툴은
- 리파지토리 마다 사용하는 툴을 바꿀 수 없다
- 여러 사람이 개발 시 사용하는 도구의 버전 지정을 강제할 수 없다
  
같은 문제가 있다.  
  
이런 문제를 해결하기 위해서, .NET Core 3.0 SDK에서는 도구를 로컬에 설치할 수 있는 시스템이 준비 되었다.    
  
좀 다르지만 npm에서는 `npm install --save-dev`이미지.  
  
  
  
### 로컬 툴을 사용
앞에 이야기 했듯이, .NET Core SDK 3.0 이후 버전을 준비한다.   
  
  
### 매니페스토 파일을 작성한다
패키지의 버전 관리를 하기 위한 `dotnet-tools.json` 파일을 작성한다.    
프로젝트의 루트에서 아래 명령을 실행한다.   
```
dotnet new tool-manifest 
```
  
실행하면 .config 폴더 내에 아래의 json 파일이 생성된다.  
.config/dotnet-tools.json  
```
{
  "version": 1,
  "isRoot": true,
  "tools": {
  }
}
```
  
  
  
### 도구를 로컬에 설치한다
`-g` 옵션을 부여하지 않으면 로컬에 설치 된다.(json 파일이 없으면 에러가 된다.)  
  
```
dotnet tool installdotnet-format 
```
  
실행 후 json 파일에 설치한 도구와 버전이 써진다.  
.config/dotnet-tools.json  
```
{
   "version": 1,
   "isRoot": true,
   "tools": {
+    "dotnet-format": {
+      "version": "3.1.37601",
+      "commands": [
+        "dotnet-format"
+      ]
+    }
   }
 }
```
  
  
## 툴 실행
실행 수단과 인수 등은 툴에 따라 다르다.  
  
```
dotnet format 
```
  
  
## 로컬 툴을 복원한다
리파지토리에서 복제한 직후 등 아직 로컬 장치를 설치하지 않을 경우 아래의 명령어를 실행한다.  
`dotnet-tools.json` 내용을 점검하고 지정된 버전의 도구가 설치된다.  
(npm에서 말하는 `npm install` 이미지)  
  
```
dotnet tool restore 
```
  
  
  
## 유용한 툴
- [dotnet-format](https://github.com/dotnet/format )
    - Lint 툴
    - 룰은 .editorconfig를 베이스로 확장([표기법](https://docs.microsoft.com/ko-kr/visualstudio/ide/editorconfig-code-style-settings-reference?view=vs-2019  ))
- [dotnet-t4](https://github.com/mono/t4 )
    - Mono기반으로 만들어진 텍스트 서식 엔진( .tt)
    - vscode에서도 T4가 가능하다
    
외에도 유용한 툴의 목록이 아래에 정리되어 있다.
https://github.com/natemcmaster/dotnet-tools 
  
  
  
## 정리
- .NET core CLI에서 툴을 사용할 수 있는 구조가 있다
- 아직 문서화 되지 않았지만, 로컬 설치도 있다.
- 유용한 툴을 사용하여 코드의 품질이나 생산성을 높이자.
  
