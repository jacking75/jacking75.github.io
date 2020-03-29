---
layout: post
title: C# - .NET Core3.1에서 CORS 설정하기
published: true
categories: [.NET]
tags: .net c# cors
---
[출처](https://qiita.com/SuyamaDaichi/items/2769c962097aacd5835d )  
  
서버 사이드와 프론트 사이드가 다른 서버로 구현된 경우, Host된 포트 번호가 다르기 때문에 CORS 설정을 할 필요가 있다.  
  
  
## 환경
- Angular 8.2.14
- .NET Core 3.1
- 서버 사이드： https://localhost:44342
- 프론트 사이드：http://localhost:4200
  
※ localhost:4200는 Angular의 기본 포트 번호  
  
  
  
## 문제
어떤 설정도 하지 않으면 프론트에서 서버 사이드에 http 요청을 한 경우 아래와 같은 에러 응답을 받는다.  
<pre>
Access to XMLHttpRequest at 'https://localhost:44342/api/weatherforecast/getweatherforecast' from origin 'http://localhost:4200' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
</pre>  
  
  
  
## Startup.cs에 CORS 설정을 한다
CORS 설정은 Startup.cs 에서 한다.  
포인트는  
- `app.UseCors();`를 `app.UseEndpoints();`의 뒤에 호출한다
- 복수의 Origin을 허가하기에는 `.WithOrigins()`에 배열로 URL을 지정한다
  
  
```
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.AspNetCore.Authentication.AzureAD.UI;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Mvc;

namespace SampleWebApi
{
    public class Startup
    {
        public Startup(IHostEnvironment env)
        {
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {

          // ~생략~

          // ↓추가↓
            services.AddCors(options =>
            {
                options.AddDefaultPolicy(
                    builder => builder
                        .AllowAnyMethod()
                        .AllowAnyHeader()
                        .WithOrigins(new string[] { "http://localhost:4200" })
                );

                options.AddPolicy("CorsPolicyName",
                     builder => builder
                        .AllowAnyMethod()
                        .AllowAnyHeader()
                        .WithOrigins(new string[] { "http://localhost:8080" })
                    );
            });

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {

          // ~생략~

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller}/{action=Index}/{id?}");
            });
          // ↓추가↓
            app.UseCors();
        }
    }
}
```
  
  
  
## API 마다 CORS를 설정하기
Controller의 속성에 `[EnableCore("CorsPolicyName")]`를 지정하면 API 마다 CORS를 설정할 수 있다.  
  
```
[HttpGet]
[Route("GetWeatherForecast")]
[Authorize]
[EnableCors("CorsPolicyName")]
public IEnumerable<WeatherForecast> Get()
{
    var rng = new Random();
    return Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = rng.Next(-20, 55),
        Summary = Summaries[rng.Next(Summaries.Length)]
    })
    .ToArray();
}
```
  
  
  