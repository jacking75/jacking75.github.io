---
layout: post
title: WCF - RESTful API
published: true
categories: [.NET]
tags: wcf .net c# RESTful
---
## RESTful 서비스
- URI나 표준 동사(POST, GET, PUT, DELETE) 등 HTTP 사양을 강하게 의식한 리소스 베이스의 접근을 제공.
- 최신의 프레임워크는 대 부분 RESTful 서비스를 만들 수 있는 기능 지원.
- WCF의 REST 서비스에서는 POX (Plan XML 형식), AJAX 애플리케이션 등에서 사용 되는 JSON (JavaScript Object Notation), RSS/Atom 등의 피드 등 다양한 메시지 포맷의 서비스를 구축하는 것이 가능.
  
  
### 샘플 코드
- Post 방식, 요청과 답변 Json
  
```
public struct REQ_DEV_ECHO
{
    public int WaitSec;
    public string ReqData;
}

public struct RES_DEV_ECHO
{
    public bool Result;
    public string ResData;
}
```
  
```
namespace WCFServerLib
{
    [ServiceContract]
    public interface IServerService
    {
        [OperationContract]
        RES_DEV_ECHO TestEcho(REQ_DEV_ECHO requestPacket);
    }
}
```
  
```
namespace WCFServerLib
{
	// 참고: "리팩터링" 메뉴에서 "이름 바꾸기" 명령을 사용하여 코드 및 config 파일에서 클래스 이름 "Service1"을 변경할 수 있습니다.
	[ServiceBehavior(InstanceContextMode = InstanceContextMode.Single, ConcurrencyMode = ConcurrencyMode.Multiple)]
	public class ServerService : IServerService
	{
		ServerService()
		{
		}

    // http://localhost:20401/Service/TestEcho
    [WebInvoke(Method = "POST",
                RequestFormat = WebMessageFormat.Json,
                ResponseFormat = WebMessageFormat.Json,
                UriTemplate = "TestEcho")]
    public RES_DEV_ECHO TestEcho(REQ_DEV_ECHO requestPacket)
    {
        var result = new RES_DEV_ECHO() { Result = true, ResData = "Hello !!!" };
        return result;
    }
}
```
  
- Post 방식, 답변은 OK
  
```
[WebInvoke(Method = "POST",
					RequestFormat = WebMessageFormat.Json,
					UriTemplate = "NotifyLockCheckUnLock")]
public void NotifyLockCheckUnLock(NTF_LS_REQUEST_UNLOCK_DATA request)
{
	try
	{
		var preCheck = ClientRequestPrepare.CheckServerStatus();
		if (preCheck != ERROR_CODE.NONE)
		{
			return;
		}

		Request.RequestLock.ProcessUnLock(request);
	}
	catch (Exception ex)
	{
		OPLogger.Exception(LOG_TYPE.REQUEST_EXCEPTION, 0, ex.Message, ex.StackTrace);
	}
}
```
  
- Post 방식, 요청과 답변은 string
  
```
[WebInvoke(Method = "POST",
                    UriTemplate = "TestRequestString")]
public string TestRequestString(string requestPacket)
{
    var result = requestPacket;
    return result;
}
```
  
![](/images/2019/wcf_restapi_post_string.png)
  
- Post 방식, 요청 application/x-www-form-urlencoded, 답변은 json
  
```
// http://localhost:20401/Service/TestRequestStream
[WebInvoke(Method = "POST",
            ResponseFormat = WebMessageFormat.Json,
            UriTemplate = "TestRequestStream")]
public RES_DEV_ECHO TestRequestStream(System.IO.Stream data)
{
    try
    {
        var reader = new System.IO.StreamReader(data);
        var result = new RES_DEV_ECHO() { Result = true, ResData = reader.ReadToEnd() };
        return result;
    }
    catch (Exception ex)
    {
        FileLogger.Exception(ex.Message);
        return new RES_DEV_ECHO() { Result = false, ResData = "" };
    }
}
```
  
![](/images/2019/wcf_restapi_post_form.png)
  
- Get 방식, 답변은 OK
  
```
[WebInvoke(Method = "GET",
					UriTemplate = "RequestHeathCheck")]
public void RequestHeathCheck()
{
	if (ServerLogic.ServerInit.EnableRequestHeathCheck)
	{
		FileLogger.Info("RequestHeathCheck");
	}
	else
	{
		OutgoingWebResponseContext response = WebOperationContext.Current.OutgoingResponse;
		response.StatusCode = System.Net.HttpStatusCode.Forbidden;
		response.StatusDescription = "Server Spot Check";
	}
}
```
  
  

## 참고
- [Support for JSONP in WCF REST services](http://www.codeproject.com/Articles/417629/Support-for-JSONP-in-WCF-REST-services)
- [WCF RESTful services and consuming them in an HTML page](http://www.codeproject.com/Articles/613097/WCF-RESTful-services-and-consuming-them-in-an-HTML)
