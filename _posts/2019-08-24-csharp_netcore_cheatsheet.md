---
layout: post
title: ASP.NET Core 애플리케이션 최소 구현 치트 시트
published: true
categories: [.NET]
tags: c# .net webapi netcore
---
[원본](https://qiita.com/taiga_takahari/items/0d8e5866519443e735e6)  
  
아래의 설정만 하면 OK.  
  
### csproj  
App.csproj  
```
<Project  Sdk = "Microsoft.NET.Sdk.Web" >

    <PropertyGroup> 
        <TargetFramework> netcoreapp2.0 </TargetFramework> 
    </PropertyGroup>

    <ItemGroup> 
        <! - 진짜 최소 구성 -> 
        <PackageReference  Include = "Microsoft.AspNetCore.Server.Kestrel"  Version = "2.0.3"  /> 
        <! - ASP.NET Core MVC (Web API)를 사용할 경우 추가 -> 
        <PackageReference  Include = "Microsoft.AspNetCore.Mvc.Core"  Version = "2.0.4"  /> 
        <! - ASP.NET Core MVC (Web Site)를 사용할 경우 추가 -> 
        <PackageReference  Include = "Microsoft.AspNetCore.Mvc"  Version = "2.0.4"  /> 
    </ItemGroup>

</Project>
```  
  
### Program.cs  
Program.cs  
```
public  class  Program 
{ 
    public  static  void  Main ( string []  args ) 
    { 
        var  builder = new  WebHostBuilder () 
            . UseKestrel()  // Kestrel을 사용 
            . UseContentRoot ( Directory.GetCurrentDirectory() )  // 프로젝트 디렉토리를 기점으로하는 
            . UseStartup<Startup>()  // Startup 구성을 로드 
            . Build();

        builder.Run(); 
    } 
}  
```  
  
### Srartup.cs  
Startup.cs  
```
public  class  Startup 
{ 
    public  IConfiguration  Configuration  {  get ;  }

    public  Startup ( IHostingEnvironment  environment ) 
    { 
        // 구성 파일과 환경 변수를 읽고 보존 
        Configuration  =  new  ConfigurationBuilder () 
            . SetBasePath( environment.ContentRootPath ) 
            . AddJsonFile( "appsettings.json" ,  false ,  true ) 
            . AddEnvironmentVariables() 
            . Build(); 
    }

    public  void  ConfigureServices ( IServiceCollection  services ) 
    { 
        // ASP.NET Core MVC (Web API) 주위의 DI 해결을 활성화 
        services.AddMvcCore(); 
        // ASP.NET Core MVC (Web Site) 주위의 DI 해결을 활성화 
        services.AddMvc (); 
    }

    public  void  Configure ( IApplicationBuilder  app ,  IHostingEnvironment  env ) 
    { 
        // HTTP 요청을 받고 HTTP 응답을 리턴한다 
        app.Run ( async  context  =>  await  context.Response.WriteAsync( "Hello World!" ));

        // ASP.NET Core MVC 라우팅을 사용 
        app.UseMvc (); 
    } 
}
```  
  
### 컨트롤러
- Web API의 경우 ControllerBase를 상속 할뿐.
    - 각 HTTP 상태 코드를 반환하는 IActionResult 메서드는 ControllerBase이다.
    - swagger 문서 생성도 ControllerBase를 찾아서 생성하고 있다.
- Web Site의 경우 Controller를 상속 할 필요가 있다.
    - 페이지를 반환하는 IActionResult는 Controller에 있다.  
  
    
### 시작
Visual Studio 2017에서 ASP.NET Core 응용 프로그램 프로젝트를 만들 때 어떤 응용 프로그램 형식을 선택해도 Microsoft.AspNetCore.All하는 패키지를 참조하는 프로젝트가 생성된다.  
  
이 Microsoft.AspNetCore.All 패키지는 상당한 양의 패키지를 모아서 정리한 메타 패키지로 실제로 이것을 참조하면 ASP.NET Core를 사용하여 Web 응용 프로그램을 개발하기 위해 필요한 것이 들어 있는데, 경우에 따라서는 불필요한 패키지의 참조도 포함되어 있기 때문에 어떤 사람들에게는 반갑지 않은 패키지로 되어 있다.  
  
또한 기본적으로 생성된 구성 설정의 소스 코드도 블랙 박스화 되어 정말 필요한 구성만 설정 되어 있는지 여부는 알기 어렵게 되어 있다.  
  
나도 불필요한 참조는 최대한 없애고 개발 하기 위때문에 Microsoft.AspNetCore.All에 의존하지 않고 최소 패키지 참조 및 구성 설정으로 개발을 시작하기 위해 메모로 남긴다.  
  
이후는 최소 구성 설정에서 서서히 기능을 추가해 나가도록 기술하고 있다.  
  
    
### Kestrel 상에서 동작하는 Web 서버를 구축하는 경우
Kestrel은 ASP.NET Core가 내포하고 있는 Web 서버를 사용하여 간단하게 HTTP 요청을 받고 HTTP 응답을 반환하는 ASP.NET Core 응용 프로그램을 구축한다.   
아마도 이 구성은 ASP.NET Core 응용 프로그램의 가장 작은 구성이 된다고 생각한다.  
  
우선 csproj 파일을 열고 Microsoft.AspNetCore.All 패키지를 삭제하고, Microsoft.AspNetCore.Server.Kestrel 패키지를 추가한다.  
  
App.csproj  
```
<ItemGroup> 
    <! - ↓를 삭제하고 ... -> 
    <! - <PackageReference Include = "Microsoft.AspNetCore.All"Version = "2.0.8"/> -> 
    <! - ↓ 추가 하기 -> 
    <PackageReference  Include = "Microsoft.AspNetCore.Server.Kestrel"  Version = "2.0.3"  /> 
</ItemGroup>
```  
  
다음으로 Program.cs을 열고 아래와 같이 코드를 다시 작성한다.  
  
Program.cs  
```
public  class  Program 
{ 
    public  static  void  Main ( string []  args ) 
    { 
        var  builder  =  new  WebHostBuilder () 
            . UseKestrel ()  // Kestrel 서버를 활성화 
            . UseStartup < Startup > ()  // Startup 클래스를 사용하는 
            . Build ();

        builder . Run (); 
    } 
}
```  
  
마지막으로 Startup.cs을 열고 코드를 아래와 같이 다시 작성한다.
  
Startup.cs  
```
public  class  Startup 
{ 
    public  void  Configure ( IApplicationBuilder  app ,  IHostingEnvironment  env ) 
    { 
        // 요청을 수락하면 "Hello World!"라고 출력하는 
        app . Run ( async  context  =>  await  context . Response . WriteAsync ( "Hello World!" ) ); 
    } 
}
```  
   
참고로 Program.cs에서 다음과 같이 기술하면 Startup.cs을 준비하지 않아도 된다.  
Program.cs  
```
public  class  Program 
{ 
    public  static  void  Main ( string []  args ) 
    { 
        var  builder  =  new  WebHostBuilder () 
            . UseKestrel () 
            // UseStartup 대신에 아래를 기술
            . Configure ( app  =>  app . Run ( async  context  =>  await  context . Response . WriteAsync ( "Hello World!" ))) 
            . Build ();

        builder . Run (); 
    } 
}
```  
   
  
## Kestrel 상에서 움직이는 ASP.NET Core MVC(Web API)를 구축
Kestrel 전용 Web 서버라면 여러 URL을 취급하므로 시간이 걸리므로, 다음은 ASP.NET Core MVC를 사용하여 Web API 서버를 구축한다.   
이것을 사용하면 URL 라우팅 기능이있는 Web API 서버 구축이 가능하다. (실제로는 더 여러가지 기능이 붙어 있지만)  
  
csproj 파일을 열고 Microsoft.AspNetCore.Mvc.Core 패키지를 추가한다.  
App.csproj  
```
<ItemGroup> 
    <PackageReference  Include = "Microsoft.AspNetCore.Mvc.Core"  Version = "2.0.4"  /> 
    <PackageReference  Include = "Microsoft.AspNetCore.Server.Kestrel"  Version = "2.0.3"  /> 
</ItemGroup >
```  
  
다음으로 Startup.cs을 다음과 같이 수정한다.  
Startup.cs  
```
public  class  Startup 
{ 
    public  void  ConfigureServices ( IServiceCollection  services ) 
    { 
        // Dependency Injection에서 ASP.NET Core MVC 주위의 기능을 해결할 수 있도록 한다 
        services . AddMvcCore (); 
    }

    public  void  Configure ( IApplicationBuilder  app ,  IHostingEnvironment  env ) 
    { 
        // ASP.NET Core MVC 라우팅을 활성화 한다 
        app . UseMvc ( route  => 
        { 
            // 우선 [컨트롤러 이름]/[ 작업 이름]으로 URL을 라우팅 
            route . MapRoute ( "default" ,  "{controller}/{action}" ); 
        }); 
    } 
}
```  
  
마지막으로 컨트롤러를 추가한다.  
이번에는 ValueController.cs 라는 파일 이름으로 하고 프로젝트 디렉토리에 생성한 Controllers 폴더에 배치한다. (프로젝트 디렉토리 아래에 배치해도 상관 없지만 나중의 유지보수를 위해 폴더를 잘라 두는 것을 추천한다.)  
ValueController.cs  
```
// Controller 대신 ControllerBase을 상속시킨다
public  class  ValueController  :  ControllerBase 
{ 
    public  string  Index () 
    { 
        return  "value" ; 
    } 
}
```  
  
  
## Kestrel 서버상에서 움직이는 ASP.NET Core MVC(Web Site) 구축
이번에는 화면이 있는 Web 애플리케이션을 구축한다.  
  
csproj 파일에서 Microsoft.AspNetCore.Mvc.Core 패키지 참조를 제거하고 Microsoft.AspNetCore.Mvc 패키지 참조를 추가한다.    
App.csproj  
```
<ItemGroup> 
    <! - ↓를 삭제하고 ... -> 
    <! - <PackageReference Include = "Microsoft.AspNetCore.Mvc.Core"Version = "2.0.4"/> -> 
    <! - ↓ 추가 -> 
    <PackageReference  Include = "Microsoft.AspNetCore.Mvc"  Version = "2.0.4"  /> 
    <PackageReference  Include = "Microsoft.AspNetCore.Server.Kestrel"  Version = "2.0.3"  /> 
</ItemGroup>
```  
  
다음으로 Startup.cs을 다음과 같이 수정한다.  
Startup.cs  
```
public  class  Startup 
{ 
    public  void  ConfigureServices ( IServiceCollection  services ) 
    { 
        // Dependency Injection에서 ASP.NET Core MVC의 View 주위의 기능을 해결할 수 있도록 한다
        services . AddMvc (); 
    }

    public  void  Configure ( IApplicationBuilder  app ,  IHostingEnvironment  env ) 
    { 
        app . UseMvc ( route  => 
        { 
            route . MapRoute ( "default" ,  "{controller}/{action}" ); 
        }); 
    } 
}
```  
  
컨트롤러와 뷰를 추가한다. 컨트롤러는 아까 만든 Controllers 폴더에 컨트롤러의 cs 파일을 배치하고, 뷰는 새로운 View 폴더를 만들고 다시 그 아래에 컨트롤러 이름과 같은 폴더를 만들고 액션 이름이 파일 이름이 되도록 cshtml 파일을 배치한다.  
  
이번에는 컨트롤러를 HomeController.cs, 뷰를 Index.cshtml 로 한다.  
HomeController.cs  
```
public  class  HomeController  :  Controller 
{ 
    public  IActionResult  Index () 
    { 
        // 액션 이름과 일치하는 View를 반환 
        return  View (); 
    } 
}
```  
  
Index.cshtml  
```
<! DOCTYPE html> 
<html> 
<body> 
    <h1> ASP.NET Core MVC </ h1> 
    <p> ASP.NET Core MVC 최소 구성 응용 프로그램입니다. </ p> 
</ body> 
</ html>
```  
  
이대로 실행하면 해당 URL로 이동해도 아무것도 표시되지 않는다.   
왜냐하면 ASP.NET Core MVC는 뷰 파일을 프로젝트 디렉토리를 기점으로 /Views/[컨트룰러 이름]/[액션 이름].cshtml 로 찾기 때문에 프로젝트 디렉토리를 명시적으로 설정할 필요가 있는 것 같다.   
프로젝트 디렉토리 설정은 Program.cs 에서 한다. Program.cs 파일을 다음과 같이 수정한다.  
Program.cs  
```
public  class  Program 
{ 
    public  static  void  Main ( string []  args ) 
    { 
        var  builder  =  new  WebHostBuilder () 
            . UseKestrel () 
            // 각 파일의 기점이되는 디렉토리를 설정 
            . UseContentRoot ( Directory . GetCurrentDirectory ()) 
            . UseStartup < Startup > () 
            . Build ();

        builder . Run (); 
    } 
}
```  
    

  
  