---
layout: post
title: C# - Http Request
published: true
categories: [.NET]
tags: c# .net HttpWebRequest
---
- 닷넷프레임워크에서는 서버로의 동시 접속수가 기본으로 최대 2로 제한되어 있다. 이 때문에 웹서비스 클라이언트에서 웹 서비스등을 호출할 때 처리량을 올리가 위해서는 멀티스레드에서 웹서비스를 호출하여도 액티브한 최대 동시 접속 수는 2로 된다.
- 싱글 스레드로 통신 할 때와 비교하면 처리량(시간당 처리량)은 향상 되지만 처리 스레드 수를 늘려도 바로 그 결과을 얻을 수 없다.
- 서버로의 동시 접속수가 최대 2로 제한된 것을 늘리기 위해서 애플리케이션 구성 파일(machine.config 파일도 가능)에 connectionManagement 요소를 아래와 같이 추가하여 최대수를 늘릴 수 있다.
  
```
<configuration>
<system.net>
  <connectionManagement>
    <add address="*" maxconnection="20"/>
  </connectionManagement>
</system.net>
</configuration>
```
  
- 보안을 고려해서 address는 제한을 해제하고 싶은 서버 URL 만을 지정하면 좋다.
  
```
<configuration>
  <system.net>
    <connectionManagement>
      <add address = “http://www.yahoo.co.jp” maxconnection = “4″ />
      <add address = “*” maxconnection = “2″ />
    </connectionManagement>
  </system.net>
</configuration>
```
  
- 설정 파일이 아닌 코드로 설정 할 수도 있다.
  
<pre>
System.Net.ServicePointManager.DefaultConnectionLimit
</pre>
  
  
  
## HttpWebRequest Post
- http://gas.gemscool.com/warnet/premium.json?mac=70-71-BC-CC-64-D6&svcCd=3ON&ip=124.137.60.50
  
``
string PostData = "mac=70-71-BC-CC-64-D6&svcCd=3ON&ip=124.137.60.50";
var httpWebRequest = (System.Net.HttpWebRequest)System.Net.WebRequest.Create(RequestUrl);
byte[] sendData = UTF8Encoding.UTF8.GetBytes(PostData);
httpWebRequest.ContentType = "application/x-www-form-urlencoded; charset=UTF-8";
httpWebRequest.Method = "POST";
httpWebRequest.ContentLength = sendData.Length;

using (var requestStream = httpWebRequest.GetRequestStream())
{
   requestStream.Write(sendData, 0, sendData.Length);
   requestStream.Close();

   var httpResponse = (System.Net.HttpWebResponse)httpWebRequest.GetResponse();

   using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
   {
       var result = streamReader.ReadToEnd();
   }
}
```
  
  
  
## HttpWebRequest Get

```
string UriRequest = string.Format("{0}/data/{1}", textBoxServerAddress.Text, RequestCount);

var httpWebRequest = (System.Net.HttpWebRequest)System.Net.WebRequest.Create(UriRequest);
httpWebRequest.Method = "GET";
httpWebRequest.ContentType = "application/x-www-form-urlencoded; charset=UTF-8";

var httpResponse = (System.Net.HttpWebResponse)httpWebRequest.GetResponse();
using (var streamReader = new System.IO.StreamReader(httpResponse.GetResponseStream()))
{
   var result = streamReader.ReadToEnd();
}
```
  
  
  
## HttpClient
- 프로젝트 참조 설정에 System.Net.Http 어셈블리를 추가, 소스 코드 선두에 System.Net.Http 이름 공간을 추가한다.
- await/async 지원.
- REST 접근을 위해 PutAsync／GetAsync／PostAsync／DeleteAsync 함수 제공.
- JSON을 사용한 Get, Post, Put, Delete
- TimeOut 시간 설정  
이 설정을 하지 않으면 처음 접속이 성공한 주소로 다시 요청을 했을 때 해당 주소에 접속을 못하면 기본 타임아웃 시간은 약 '30'초 정도.  
서버에서 사용할 때는 타임아웃 시간을 아래처럼 설정하는 것이 좋다.  
**HttpClient httpClient = new HttpClient() { Timeout = TimeSpan.FromMilliseconds(1200) };**  
  
```
namespace Codeplex.HttpClientExtensions
{
    using Newtonsoft.Json;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using System.Threading.Tasks;

    public static class HttpClientExtensions
    {
        public static Task<HttpResponseMessage> PostAsJsonAsync<T>(this HttpClient self, string uri, T obj)
        {
            var content = CreateHttpContentFromObject(obj);
            return self.PostAsync(uri, content);
        }

        public static Task<HttpResponseMessage> PutAsJsonAsync<T>(this HttpClient self, string uri, T obj)
        {
            var content = CreateHttpContentFromObject(obj);
            return self.PutAsync(uri, content);
        }

        public static async Task<T> ReadAsJsonAsync<T>(this HttpContent content)
        {
            var binary = await content.ReadAsByteArrayAsync();
            var jsonText = Encoding.UTF8.GetString(binary, 0, binary.Length);
            return JsonConvert.DeserializeObject<T>(jsonText);
        }

        private static HttpContent CreateHttpContentFromObject(object obj)
        {
            var jsonText = JsonConvert.SerializeObject(obj);
            var content = new ByteArrayContent(Encoding.UTF8.GetBytes(jsonText));
            content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
            return content;
        }
    }
}


var root = "http://localhost:9454/api/";

var c = new HttpClient();
{
    // Get
    var r = await c.GetAsync(root + "values");
    r.EnsureSuccessStatusCode();
    var ps = await r.Content.ReadAsJsonAsync<IEnumerable<Person>>();
    foreach (var p in ps)
    {
        Console.WriteLine("{0} {1}", p.Id, p.Name);
    }
}

{
   // Post
   await c.PostAsJsonAsync(root + "values", new Person { Id = -1, Name = "Tester" });

   // Put
   await c.PostAsJsonAsync(root + "values/10", new Person { Id = 10, Name = "Tester" });
}
```
  
- query 파라미터를 입력으로 사용하는 Get
query 파라미터는 http://localhost:9454/api/id="dd"&pw="dd" 식으로 값을 입력하는 방식을 뜻한다.
  
```
  var httpClient = new HttpClient();

  var content = new FormUrlEncodedContent(new Dictionary<string, string>
  {
      { "foo", "111" },
      { "bar", "222" },
      { "baz", "333" },
  });

  var response = await httpClient.PostAsync("http://localhost/", content);
```
  
- query 파라미터를 입력으로 사용하는 Post
  
```
  using System.Net.Http;

  class Program
  {
      static void Main()
      {
          using (var client = new HttpClient())
          {
              client.BaseAddress = new Uri("http://localhost:6740");
              var content = new FormUrlEncodedContent(new[]
              {
                  new KeyValuePair<string, string>("", "login")
              });
              var result = client.PostAsync("/api/Membership/exists", content).Result;
              string resultContent = result.Content.ReadAsStringAsync().Result;
              Console.WriteLine(resultContent);
          }
      }
  }
```
  
- 델리게이트 사용하기
    - HttpClient 생성자에 HttpMessageHandler를 구현한 클래스를 지정하여 이 HttpClient 클래스를 사용하여 HTTP 요청에 대해 독자적인 처리를 하도록 한다.
    - 보통 DelegationHandler를 상속하여 SendAsync 메소드를 오버라이드 한다.
  
```
public class MyMessageHandler : DelegationHandler
{
    public override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        // HttpRequestMessage를 사용해서 요청을 한다.

        // 베이스 클래스의 SendAsync를 호출한다.
        var response = base.SendAsync(request, cancellationToken);

        return response;
    }
}

var httpClient = new HttpClient(new MyMessageHandler());
```
  
- HttpClinet 사용 예
  
```
public class NetHttpRequester
{
    public static async Task<HttpResponseMessage> Request<T>(string apiUrl, T requestData)
    {
        var content = CreateHttpContentFromObject(requestData);

        HttpClient httpClient = new HttpClient();
        var response = await httpClient.PostAsync(apiUrl, content);
        return response;
    }

    static HttpContent CreateHttpContentFromObject(object obj)
    {
        var jsonRedisValueConverter = new CloudStructures.Redis.JsonRedisValueConverter();
        var jsonText = jsonRedisValueConverter.Serialize(obj);
        var content = new ByteArrayContent(jsonText);
        content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        return content;
    }
}

// 사용
var request = new REQ_LS_REQUEST_LOCK_DATA()
{
    SN = sn,
    UserID = userID,
    Token = authToken,
};


var response = await NetHttpRequester.Request<REQ_LS_REQUEST_LOCK_DATA>(apiUrl, request);
if (response.IsSuccessStatusCode == false)
{
    FileLogger.Error(string.Format("Netowrk Error RequestLockCheck. {0}", apiUrl));
    return ERROR_CODE.REQUEST_LOCK_NETWORK_ERROR;
}

var result = await response.Content.ReadAsStringAsync();
var jsonResObject = Jil.JSON.Deserialize<RES_LS_REQUEST_LOCK_DATA>(result);

if (jsonResObject.Result != (short)ERROR_CODE.NONE)
{
    return (ERROR_CODE)jsonResObject.Result;
}
```
  
- Extending HttpClient with OAuth to Access Twitter [http://code.msdn.microsoft.com/Extending-HttpClient-with-8739f267]
- [HttpClient을 사용하여 POST로 멀티 파트의 파일을 전송하기](http://qiita.com/volpe28v@github/items/b86e0bc9db8e42688cc3)
- [C# HttpClient로 인증된 프록시를 이용하기](http://qiita.com/hugo-sb/items/6611ae2aa24635429726 )
  
  
  
## HttpWebRequest : 파일 전송하기
- [http://crystalcube.co.kr/20]
- [http://blog.naver.com/9swon/100021144866]
  
  
  
## RestSharp - Simple REST and HTTP Client for .NET
- 오픈소스 http 클라이언트 라이브러리
- 공식 사이트 http://restsharp.org/
- GitHub  https://github.com/restsharp/RestSharp
- 소개  http://pawel.sawicz.eu/restsharp/
- 클라이언트에서만 사용하는 것이 좋다는 말이 있음. [RestSharp 라이브러리의 CPU High 현상](http://www.sysnet.pe.kr/Default.aspx?mode=2&sub=0&detail=1&pageno=0&wid=10868&rssMode=1&wtype=0)
- 예제  http://stackoverflow.com/questions/10226089/restsharp-simple-complete-example
  
```
// 답변 숫자_숫자
var client = new RestSharp.RestClient(ServerConfig.LogServerURI);
string RestAPI = string.Format("{0}/{1}", ServerConfig.UniqueNumberRequestAPI, ServerConfig.MaxUniqueNumberStockCount);
var request = new RestSharp.RestRequest(RestAPI, RestSharp.Method.GET);
var queryResult = client.Execute<List<string>>(request).Data;

var TokenString = queryResult[0].Split(new Char[] { '_' }, StringSplitOptions.RemoveEmptyEntries);
```
  
  
  
## Refit: The automatic type-safe REST library for Xamarin and .NET
- RestSharp이나 HttpClient와 같은 Http 통신 관련 오픈 소스 라이브러리
- Java의 retrofit(http://square.github.io/retrofit/ )에서 영감을 받아서 만들었음.
- 특징은 쉽게 사용할 수 있고, 비동기 지원.
- 지원하는 플랫폼
    - Xamarin.Android
    - Xamarin.Mac
    - Desktop .NET 4.5
    - Windows Phone 8
    - Silverlight 5
    - Xamarin.iOS            (예정)
    - Windows Store (WinRT)  (예정)
- 공식 사이트: http://paulcbetts.github.io/refit/
- 소스 코드: https://github.com/paulcbetts/refit
- NuGet으로 얻을 수 있다.
    - NuGet으로 설치 후 라이브러리 참조에 System.Net.Http가 없다면 직접 추가해야 한다.
- 예제1)
  
```
public interface IGitHubApi
{
    [Get("/users/{user}")]
    Task<User> GetUser(string user);
}

var gitHubApi = RestService.For<IGitHubApi>("https://api.github.com");
var octocat = await gitHubApi.GetUser("octocat");
```
  
- 예제2) JSon 데이터를 Post로 보내고 받는 경우
  
```
public interface IGameServiceRatesApi
{
    [Post("/GameService/Login")]
    Task<RES_LOGIN> Login([Body] REQ_LOGIN request);
}

public class REQ_LOGIN
{
    public string ID;
    public string PW;
}

public class RES_LOGIN_DATA
{
    public short Result;
    public string AuthToken;
}


var serverRequest = RestService.For<IGameServiceRatesApi>("http://12.20.10.17:13501");

var requestLogin = new REQ_LOGIN
{
    ID = "test1",
    PW = "test1",
};

try
{
    var result = serverRequest.Login(requestLogin);
    result.Wait();
    Console.WriteLine(result.ToString());
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```
  