---
layout: post
title: C# - ScriptCs.WebApi
published: true
categories: [.NET]
tags: c# .net ScriptCs WebApi
---
## 개요
- 2015.09.14  현재 WebAPI 2는 ScriptCs 1.5에서 에러 발생
- [ScriptCs.WebApi2](https://github.com/scriptcs-contrib/scriptcs-webapi)
- WebApi를 사용할 수 있게 해준다.
  
  
  
## 설치 및 간단 사용
- Web API script pack을 설치한다.
    - scriptcs -install ScriptCs.WebApi2
- start.csx 파일을 만든 후 아래의 내용을 입력한다
  
```
using System.Dynamic;

public class TestController : ApiController
{
    public dynamic Get() {
        dynamic obj = new ExpandoObject();
        obj.message = "Hello from Web Api";
        return obj;
    }
}

var webapi = Require<WebApi>();

var server = webapi.
    Configure(typeof(TestController)).
    UseJsonOnly().
    Start("http://localhost:8080");

Console.WriteLine("Listening...");
Console.ReadLine();
server.Dispose();
```
  
- 실행
    - Windows:  scriptcs start.csx -modules mono
    - Linux/Mac: scriptcs start.csx
- 브라우져를 실행 후 아래의 주소를 입력한다.
    - http://localhost:8080/api/test
  
  
  
## Customizing
- You can configure the OWIN host by passing in an Action<IAppBuilder> to the Configure method
  
```
var server = webapi.
    Configure(
        typeof(TestController),
        builder=> {
          builder.Use<MyMiddleware>();
        }
    ).
    Start("http://localhost:8080");
```
  
  
  
## 독자 포맷 사용
- [Media Formatters in ASP.NET Web API 2](http://www.asp.net/web-api/overview/formats-and-model-binding/media-formatters)
- [BinaryMediaTypeFormatter](http://byterot.blogspot.kr/2012/04/aspnet-web-api-series-part-5.html)
  
```
using System.Dynamic;
using System.Web.Http;
using System.Web.Http.Routing;

public class TestController : ApiController
{
	public dynamic Get() {
		dynamic obj = new ExpandoObject();
		obj.message = "Hello from Web Api";
		return obj;
	}
}

var webapi = Require<WebApi>();

var formatter = webapi.NewFormatter().
	SupportMediaType("application/vnd.foo+json").
	MapUriExtension(".foo", "application/vnd.foo+json").
	WriteToStream(async (args) => {
		var writer = new StreamWriter(args.Stream);
		await writer.WriteLineAsync("{\"foo\":\"foo\"}");
		await writer.FlushAsync();
	}).
	Build();

var config = new HttpConfiguration();

webapi.
	UseJsonOnly().
	Configure(config, typeof(TestController));

config.Formatters.Insert(0, formatter);

config.Routes.Clear();
config.Routes.MapHttpRoute(name: "Extension",
	routeTemplate: "api/{controller}.{extension}/{id}",
	defaults: new {id = RouteParameter.Optional}
);

var server = webapi.Start("http://localhost:8080");

Console.WriteLine("Listening...");
Console.ReadLine();
server.Dispose();
```
  
  
  
## 속성 라우팅
- http://localhost:8080/api/Test/hello
  
```
using System.Dynamic;

public class TestController : ApiController
{
	[Route("api/Test/hello")]
	public dynamic Get() {
        dynamic obj = new ExpandoObject();
        obj.message = "Hello from Web Api";
        return obj;
    }
}


var webapi = Require<WebApi>();

var server = webapi.
    Configure(typeof(TestController)).
    UseJsonOnly().
    Start("http://localhost:8080");

Console.WriteLine("Listening...");
Console.ReadLine();
server.Dispose();
```
  
  
  
## query string
  
```
using System.Dynamic;

public class TestController : ApiController
{
	//http://localhost:8080/api/Test?bbsId=G003&itemId=5
	public dynamic Post(string bbsId, int itemId) {
		dynamic obj = new ExpandoObject();
        obj.message = "ddd";
        return obj;
    }
}


var webapi = Require<WebApi>();

var server = webapi.
	Configure(typeof(TestController)).
    Start("http://localhost:8080");

Console.WriteLine("Listening...");
Console.ReadLine();
server.Dispose();
```
  
  
  
## 복수의 컨트룰러 사용
  
```
using System.Dynamic;

public class TestController : ApiController
{
	[Route("api/Test/hello")]
	public dynamic Get() {
        dynamic obj = new ExpandoObject();
        obj.message = "Hello from Web Api";
        return obj;
    }
}

public class Test2Controller : ApiController
{
	[Route("api/Test2/hello")]
	public dynamic Get() {
        dynamic obj = new ExpandoObject();
        obj.message = "Hello from Web Api 2";
        return obj;
    }
}

var webapi = Require<WebApi>();

var server = webapi.
    Configure(typeof(TestController), typeof(Test2Controller)).
    UseJsonOnly().
    Start("http://localhost:8080");

Console.WriteLine("Listening...");
Console.ReadLine();
server.Dispose();
```
  
  
  
## 참고
- [(일어)ASP.NET Web API 2 Attribute Routing](http://miso-soup3.hateblo.jp/entry/2013/07/03/223816)
- [(일어)ASP.NET Web API attribute 라우팅에 대해서](http://katoj.hatenablog.com/entry/2015/04/24/195153)
- [(일어)ASP.NET Web API 입문](http://www.atmarkit.co.jp/ait/subtop/features/dotnet/aspwebapi_index.html)
