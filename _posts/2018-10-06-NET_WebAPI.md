---
layout: post
title: 최소한으로 Web API 사이트 만들기
published: true
categories: [.NET]
tags: .net c# asp.net webapi
---
* Visual Studio의 프로젝트 템플릿으로 만들면 불필요한 코드가 너무 많이 붙어 있음.
    * 특히 Razor은 사용하지 않고 클라이언트 부분은 JavaScript를 사용하고 싶은 경우에 좋지 않음.
* Building Out a Clean, REST-ful WebAPI Service with a Minimal WebApi Project 라는 글을 참고하면 최소한의 구성으로 만들 수 있음  
http://www.codeproject.com/Articles/615804/Building-Out-a-Clean-REST-ful-WebAPI-Service-with  
    * 템플릿으로 만든 것을 받아서 시작할 수도 있음
    https://github.com/xivSolutions/EmptyWebApiProject
    * 또는 위의 것을 zip 파일로 된 것을 받을 수 있음
    https://github.com/xivSolutions/EmptyWebApiProject/archive/master.zip
* 전체적인 흐름은 
    * Controllers에 빈 API 컨트롤러를 만들고 
    * App_Start 폴더의 WebApiConfig.cs 파일을 아래처럼 한다.
	
```
public const string DEFAULT_ROUTE_NAME = "MyDefaultRoute";

public static void Register(HttpConfiguration config)
{
   config.Routes.MapHttpRoute(
	  name: DEFAULT_ROUTE_NAME,
	  routeTemplate: "api/{controller}/{id}",
	  defaults: new { id = RouteParameter.Optional }
   );

   config.EnableSystemDiagnosticsTracing();
}
```

    * Global.asax 파일에는 아래처럼 한다.

```
public class WebApiApplication : HttpApplication
{
   protected void Application_Start()
   {
	  AreaRegistration.RegisterAllAreas();

	  EmptyWebApiProject.App_Start.WebApiConfig.Register(GlobalConfiguration.Configuration);
	  EmptyWebApiProject.App_Start.FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
   }
}
```
    * NuGet으로 아래의 어셈블리 설치한다
    * NuGet으로 JQuery와 같은 스크립트 라이브러리도 설치한다
* api 주소 명시적으로 할당하기
    * 암시적으로 주소를 할당하는 것 보다 명시적으로 하는 것이 좋을 듯. 다만 언제나 암시적인 것이 우선 되므로 암시적으로 할당 되지 않도록 api 함수 이름 지정을 조심하도록!
    * 주소 지정 방식은 여기를 참고 바람  
    http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2

```
// 주소 지정 방식 http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2

public class GameItemController : ApiController
{
	// http://localhost:9000/api/GameItem/alllist?version=1
	[HttpGet, ActionName("alllist/{version:int}")]
	public string AllGameItemList(int version)
	{
		var itemlist = MongoDBWork.GetItemList(version);
		string output = Newtonsoft.Json.JsonConvert.SerializeObject(itemlist, Newtonsoft.Json.Formatting.None).Trim();
		return output;
	}

	// http://localhost:9000/api/GameItem/AddItem  JSon으로 item 데이터를 보낸다.
	[HttpPost, ActionName("AddGameItem/item")]
	public HttpResponseMessage AddGameItem(ItemInfo item)
	{
		var response = Request.CreateResponse<ItemInfo>(HttpStatusCode.Created, item);
		string uri = Url.Link(EmptyWebApiProject.App_Start.WebApiConfig.DEFAULT_ROUTE_NAME, new { id = "OK" });
		response.Headers.Location = new Uri(uri);
		return response;
	}
}
```
  
  

### 참조
* [공식 사이트](http://www.asp.net/web-api) 
* [1-1 당신의 첫번째 ASP.NET Web API ( C# )](http://blog.naver.com/wow0815/90157342752) 
* [1-2 당신의 첫번째 ASP.NET Web API ( C# )](http://blog.naver.com/wow0815/90157426005) 
* [ASP.NET MVC3 MVC4 게시 및 배포 ( 403.14 에러 조치법 )](http://blog.naver.com/wow0815/90157586668) 
* [2-1 CRUD 작업을 지원하는 Web API 만들기!](http://blog.naver.com/wow0815/90157998418) 
* [ASP.NET MVC Web API JSONP](http://getchar.tistory.com/39) 
    * http://blog.naver.com/catter97/167705519
