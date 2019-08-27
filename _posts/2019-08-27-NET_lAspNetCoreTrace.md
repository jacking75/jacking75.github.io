---
layout: post
title: ASP.NET Core 서버 시작까지 흐름 추적
published: true
categories: [.NET]
tags: .net c# .netcore aspnetcore
---
[출처](https://qiita.com/sengoku/items/9ae4bfd43f6660abb935 )  

## 프로그램 엔터리

### Program#Main
Program 클래스의 Main 메소드 안은 아래처럼 되어 있다.  
  
이것을 1행씩 따라가 본다.  
```
Program.cs
WebHost.CreateDefaultBuilder(args)
    .UseStartup<Startup>()
    .Build()
    .Run();
```
  
  
## 1행: .CreateDefaultBuilder(args)

### WebHostBuilder 인스턴스 만들기
WebHostBuilder 인스턴스를 만든다.  

소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/DefaultBuilder/src/WebHost.cs  
```
var builder = new WebHostBuilder();
```
  
  
### 환경 변수를 읽도로 한다.  
WebHostBuilder 생성자의 주요 부분이다.  

ConfigurationBuilder의 확장 메소드를 호출하고, 프리픽스 ASPNETCORE_ 가 붙은 환경 변수를 읽는다.  

소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/WebHostBuilder.cs
  
```  
_hostingEnvironment = new HostingEnvironment();

_config = new ConfigurationBuilder()
    .AddEnvironmentVariables(prefix: "ASPNETCORE_")
    .Build();

_context = new WebHostBuilderContext
{
    Configuration = _config
};
```
  
ConfigurationBuilder#AddEnvironmentVariables  
  
지정된 프리릭스가 붙은 환경 변수를 읽도록 한다.  

소스 파일: https://github.com/aspnet/Extensions/blob/master/src/Configuration/Config.EnvironmentVariables/src/EnvironmentVariablesExtensions.cs  

```
public static IConfigurationBuilder AddEnvironmentVariables(
    this IConfigurationBuilder configurationBuilder,
    string prefix)
{
    configurationBuilder.Add(new EnvironmentVariablesConfigurationSource { Prefix = prefix });
    return configurationBuilder;
}
```
  
  
### 문서 루트를 현재 디렉토리로 한다

```
if (string.IsNullOrEmpty(builder.GetSetting(WebHostDefaults.ContentRootKey)))
{
    builder.UseContentRoot(Directory.GetCurrentDirectory());
}
```
  
  
### 명령어 라인 인수를 처리
dotnet run 명령어에서는 아무 것도 넘기지 않으므로 실질적으로 아무 것도 하지 않는다.    

src/DefaultBuilder/src/WebHost.cs  
```
if (args != null)
{
    builder.UseConfiguration(new ConfigurationBuilder().AddCommandLine(args).Build());
}
```  
  
  
### 설정 로딩
환경에 따라서 설정 파일을 읽는 delegate를 WebHostBuilder 에 넘기고 있다.  

default로는 appsettings.json 파일을 읽는다.  
또 appsettings.{환경}.json 파일을 읽는다.  

Development 환경 때는 추가 어셈블리를 읽는다.  
  
src/DefaultBuilder/src/WebHost.cs  
```
builder.ConfigureAppConfiguration((hostingContext, config) =>
{
    var env = hostingContext.HostingEnvironment;

    config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true);

    if (env.IsDevelopment())
    {
        var appAssembly = Assembly.Load(new AssemblyName(env.ApplicationName));
        if (appAssembly != null)
        {
            config.AddUserSecrets(appAssembly, optional: true);
        }
    }

    config.AddEnvironmentVariables();

    if (args != null)
    {
        config.AddCommandLine(args);
    }
})
```
  
### logger 설정
logger를 설정한다.
  
```
.ConfigureLogging((hostingContext, logging) =>
// （생략）
)
``` 
  
  
### default 서비스 프로바이더를 설정

```
.UseDefaultServiceProvider((context, options) =>
{
    options.ValidateScopes = context.HostingEnvironment.IsDevelopment();
})
```
  
   
   
### default 설정
Kestrel web 서버를 사용한다
  
```
.UseKestrel((builderContext, options) =>
{
    options.Configure(builderContext.Configuration.GetSection("Kestrel"));
})
```
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Servers/Kestrel/Kestrel/src/WebHostBuilderKestrelExtensions.cs  
  
  
### DI 컨테이너에 추가

```
.ConfigureServices((hostingContext, services) =>
{
    services.PostConfigure<HostFilteringOptions>(/* （생략） */);

    services.AddSingleton<IOptionsChangeTokenSource<HostFilteringOptions>>(/* （생략） */);

    services.AddTransient<IStartupFilter, HostFilteringStartupFilter>();

    services.AddRouting();
})
```
  
  
### IIS를 사용한다(?)

```
.UseIIS()
.UseIISIntegration()
```
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Servers/IIS/IIS/src/WebHostBuilderIISExtensions.cs  
  
  
  
## 2행: .UseStartup<Startup>()
DI 컨테이너에 StartUp 클래스를 등록하는 delegate를 WebHostBuilder에 추가한다.  
  
  
### 실제로는 StartUp 클래스 대신에 ConverntionBasedStartup 클래스를 등록  
실제로는 ConverntionBasedStartup 클래스를 사용하도록 한다  

소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/WebHostBuilderExtensions.cs  
```
return hostBuilder
    .ConfigureServices(services =>
    {
// (생략)
        services.AddSingleton(typeof(IStartup), sp =>
        {
            var hostingEnvironment = sp.GetRequiredService<IHostingEnvironment>();
            return new ConventionBasedStartup(StartupLoader.LoadMethods(sp, startupType, hostingEnvironment.EnvironmentName));
        });
// (생략)
    });
```
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/Startup/ConventionBasedStartup.cs  
  
  
### StartUp 클래스 호출처 메소드를 결정
ConverntionBasedStartup 클래스를 호출하는 메소드는 StartupLoader에서 결정한다.  
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/Internal/StartupLoader.cs  
  
  
  
## 3행: .Build()
WebHost를 빌드한다.  

소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/WebHostBuilder.cs  
  
  
### 공통 서비스 등록

```
var services = new ServiceCollection();
// （생략）
services.AddSingleton<IHostingEnvironment>(_hostingEnvironment);
services.AddSingleton<Extensions.Hosting.IHostingEnvironment>(_hostingEnvironment);
services.AddSingleton(_context);

// （생략）

var configuration = builder.Build();
services.AddSingleton<IConfiguration>(configuration);
_context.Configuration = configuration;

var listener = new DiagnosticListener("Microsoft.AspNetCore");
services.AddSingleton<DiagnosticListener>(listener);
services.AddSingleton<DiagnosticSource>(listener);

services.AddTransient<IApplicationBuilderFactory, ApplicationBuilderFactory>();
services.AddTransient<IHttpContextFactory, DefaultHttpContextFactory>();
services.AddScoped<IMiddlewareFactory, MiddlewareFactory>();
services.AddOptions();
services.AddLogging();

// （생략）
```
  
  
### WebHost 인스턴스를 만든다
WebHost 인스턴스를 만들고, Initialize를 호출하고 있다.  

```
var host = new WebHost(
    applicationServices,
    hostingServiceProvider,
    _options,
    _config,
    hostingStartupErrors);

(snip)
host.Initialize();
```
  
  
### WebHost 초기화
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/Internal/WebHost.cs  
  
StartUp 클래스의 ConfigureServices를 호출  
```
_startup = _hostingServiceProvider.GetService<IStartup>();
```
  
```
_applicationServices = _startup.ConfigureServices(_applicationServiceCollection);
```
  
  
  
## 4행: .Run();

### WebHost의 StartAsync를 호출
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/WebHostExtensions.cs  
```
await host.StartAsync(token);
```
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/Internal/WebHost.cs  
  
  
### 어플리케이션 빌드

```
var application = BuildApplication();
```
  
  
#### 서버 만들기

```
Server = _applicationServices.GetRequiredService<IServer>();
```
  
  
#### 어드레스 추가

```
var urls = _config[WebHostDefaults.ServerUrlsKey] ?? _config[DeprecatedServerUrlsKey];
// （생략）
foreach (var value in urls.Split(new[] { ';' }, StringSplitOptions.RemoveEmptyEntries))
{
    addresses.Add(value);
}
```
  
  
#### ApplicationBuilder 만들기

```
var builderFactory = _applicationServices.GetRequiredService<IApplicationBuilderFactory>();
var builder = builderFactory.CreateBuilder(Server.Features);
builder.ApplicationServices = _applicationServices;
```
  
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Hosting/Hosting/src/Builder/ApplicationBuilderFactory.cs  

```
return new ApplicationBuilder(_serviceProvider, serverFeatures);
```
  
   
#### 필터 클래스들을 호출

```
var startupFilters = _applicationServices.GetService<IEnumerable<IStartupFilter>>();
Action<IApplicationBuilder> configure = _startup.Configure;
foreach (var filter in startupFilters.Reverse())
{
    configure = filter.Configure(configure);
}

configure(builder);
```
  
  
#### 어플리케이션 빌드
소스 파일: https://github.com/aspnet/AspNetCore/blob/master/src/Http/Http/src/Internal/ApplicationBuilder.cs  

_components 에는 use한 delegate가 들어가 있다.  

```
// （생략）
foreach (var component in _components.Reverse())
{
    app = component(app);
}

return app;
```
  
  
### HostingApplication 인스턴스 만들기

```
var diagnosticSource = _applicationServices.GetRequiredService<DiagnosticListener>();
var httpContextFactory = _applicationServices.GetRequiredService<IHttpContextFactory>();
var hostingApp = new HostingApplication(application, _logger, diagnosticSource, httpContextFactory);
```
  
  
#### 서버 기동

```
await Server.StartAsync(hostingApp, cancellationToken).ConfigureAwait(false);
```
  
