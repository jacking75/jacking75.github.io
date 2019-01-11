---
layout: post
title: WCF - AsyncAwait
published: true
categories: [.NET]
tags: wcf .net c# AsyncAwait
---
예제 코드  
```
[ServiceContract]
public interface IServerService
{
	[OperationContract]
	Task<RES_LOGIN> RequestUserLogin(REQ_LOGIN reqData);
}		

// http://localhost:15501/GameService/RequestUserLogin
[WebInvoke(Method = "POST",
			RequestFormat = WebMessageFormat.Json,
			ResponseFormat = WebMessageFormat.Json,
			UriTemplate = "RequestUserLogin")]
public async Task<RES_LOGIN> RequestUserLogin(REQ_LOGIN reqData)
{
	try
	{
		var result = await ReqLogin.ProcessAsync(reqData);
		return result;
	}
	catch (Exception ex)
	{
		return new RES_LOGIN { Result = ERROR_CODE.SERVER_ERROR };
	}
}

public static async Task<RES_LOGIN> ProcessAsync(REQ_LOGIN reqData)
{
	var 메모리DB_인증성공 = await CheckAuthAsync(reqData.ID, null, reqData.PW);

	if (메모리DB_인증성공 == null)
	{
		var dbResult = await Task.Run(() => GetUserAuth(reqData.ID, hashPW));

		if (dbResult == null)
		{
			responseResult.Result = ERROR_CODE.AUTH_FAIL_ID_PW;
			return responseResult;
		}

		var dbUserData = await Task.Run(() => GetMainData(dbResult.UserIndex));

		SaveUserAuthInfo(ref responseResult, reqData, dbResult, dbUserData);

		return responseResult;
	}

	return responseResult;
}
```  
  
  
- [(일어)ASP.NET WebForms 4.5, WCF 4.5 에서 비동기(async) 메소드](http://blogs.msdn.com/b/tsmatsuz/archive/2012/05/28/net-4-5-asp-net-webforms-and-wcf-async-method.aspx)
- [(영어)Asynchronous Operations in WCF](http://jaliyaudagedara.blogspot.kr/2013/03/asynchronous-operations-in-wcf.html)
- [(영어)Async WCF Today and Tomorrow](http://blog.stephencleary.com/2012/08/async-wcf-today-and-tomorrow.html)
  