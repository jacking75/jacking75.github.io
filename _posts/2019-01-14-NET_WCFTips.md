---
layout: post
title: WCF - Tips
published: true
categories: [.NET]
tags: wcf .net c#
---
### 클라이언트가 요청 데이터를 null로 보내는 경우 대처
클라이언트에서 public RES_LOGIN_DATA RequestLogin(REQ_LOGIN_DATA reqData)을 요청한 경우 reqData을 null로 보낼 수 있음(악의적인 이유로).  
그래서 아래처럼 요청 데이터를 null 조사를 해야한다.  
```
public class ServerService : IServerService
{
	public RES_LOGIN_DATA RequestLogin(REQ_LOGIN_DATA reqData)
	{
		if (reqData == null) // 처럼 null 값 조사를 해야 됨
		{
			return new RES_LOGIN_DATA { Result = ERROR_CODE.SERVER_ERROR };
		}
```
  
  
### Session과 바인딩
- Session 지원 바인딩
    - NetTcpBinding, WSHttpBinding, NetNamedPipeBinding, etc, every proxy gets an instance of the service. 
- Session 미지원 바인딩 
    - BasicHttpBinding, WebHttpBinding, we get the same effect as PerCall. 
  
  
  
### WCF Instances and Threading
http://www.dotnetconsult.co.uk/weblog2/2011/02/04/WCFInstancesAndThreading.aspx  
  
  
### 에러 답변 보내기
- AWS의 경우 로드밸런스(ELB)에 특정 API를 알려주면 정기적으로 요청을 하고 그 요청에 대해서 OK 답변을 주면 ELB는 해당 서버가 살아 있다고 본다.
  
``` 
// http://localhost:21301/Service/RequestHeathBeat
[WebInvoke(Method = "GET", UriTemplate = "RequestHeathBeat")]
public void RequestHeathBeat()
{
}
```
  
  
- 그러나 서버 점검 등을 이유로 ELB에 에러 값을 통보 하고 싶은 경우는 아래처럼 한다.
  
```
// http://localhost:21301/Service/RequestHeathBeat
[WebInvoke(Method = "GET", UriTemplate = "RequestHeathBeat")]
public void RequestHeathBeat()
{
    OutgoingWebResponseContext response = WebOperationContext.Current.OutgoingResponse;
    response.StatusCode = System.Net.HttpStatusCode.Forbidden;
    response.StatusDescription = "Server Spot Check";
}
```
  
  
  
### 클라이언트 IP 알아내기
- WCF 라이브러리 부분에서 사용할 수 있다.
  
```
private string GetIP()
{
   var context = OperationContext.Current;
   var mp = context.IncomingMessageProperties;
   var propName = System.ServiceModel.Channels.RemoteEndpointMessageProperty.Name;
   var prop = (System.ServiceModel.Channels.RemoteEndpointMessageProperty)mp[propName];
   string remoteIP = prop.Address;
   return remoteIP;
}
```
  