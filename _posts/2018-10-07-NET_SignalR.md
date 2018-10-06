---
layout: post
title: SignalR 정리
published: true
categories: [.NET]
tags: .net c# asp.net SignalR
---
### SignalR 정리
http://jacking.tistory.com/1139  
  

### self 호스팅
* (영어) http://www.asp.net/signalr/overview/signalr-20/getting-started-with-signalr-20/tutorial-signalr-20-self-host
* (일어. 클라이언트) http://okazuki.hatenablog.com/entry/20130512/1368371469
* 위의 도움말과 다르게 Nuget으로 SignalR-selfHost을 설치한 후 따로 Microsoft.OWIN.Cors를 Nuget에서 다시 설치해야 한다.
* 서버의 경우 SignalR 부분을 클래스 라이브러리에서 사용하는 경우 이 클래스 라이브러리를 사용하는 프로그램에서 class Startup을 재정의 해야한다. Startup 클래스는 프로그램에서만 로딩하는 것 같다(2013.11.05)
  
  

### rpc 함수 이름 한글 사용
* 메시지 이름은 당연(?) 영어가 아닌 한글도 가능하다.  
```
// 서버측 리시브 부분
public void 테스트_메시지보내기(string message)
{
   Clients.All.테스트_메시지보내기(string.Format("Re:{0}. {1}", message, DateTime.Now));
}
// 클라이언트측 보내기 부분
public void 테스트_메시지보내기(string msg)
{
   agentHubProxy.Invoke("테스트_메시지보내기", msg).Wait();
}
```  
  

### WebApp.Start 호출하기
* using 문에서 Dispose 호출하기  
```
// 스레드에서 호출하는 함수
public void NetworkThreadProc()
{
    using (WebApp.Start(serverAddress))
    {
        while (IsThreadRunning)
        {
            ...
        }
    }
}
```  
  
* 명시적으로 Dispose 호출하기  
```
IDisposable SignalR;
....
SignalR = WebApp.Start(serverAddress);
....
SignalR.Dispose();
```  
  

### 특정 Hub 인스턴스를 얻기

```
_hubContext = GlobalHost.ConnectionManager.GetHubContext<MoveShapeHub>();
```
  

### Hub 인스턴스의 Clients 얻기

```
GlobalHost.ConnectionManager.GetHubContext<StockTickerHub>().Clients
```  
  

### 서버 주소 지정
* Self-Host에서는 로컬 접속에서는 "http://localhost:9080" 라고 설정하면 
    * 클라이언트 접속이 잘 되지만 클라이언트가 리모트로 접속할 때는 접속 거부를 한다.
    * 서버에서 리모트에 있는 클라이언트의 접속을 받아 들이기 위해서는 주소를 다음과 같이 한다. 
    "http://*:9080"
    그리고 위와 같이 주소를 지정할 때는 관리자 권한으로 실행해야 한다.
  

### 데이터를 보낼 때는 비동기로 보낸다
* Windows 7에서는 문제 없으나 Invoke 호출 후 Wait()를 호출하면 Windows Server 2012에서는 먹통이 된다.  
```
HubProxy.Invoke("서버 함수", json).Wait(); // 이렇게 하면 먹통이 된다.
HubProxy.Invoke("서버 함수", json)         // 이렇게 해야 한다.
```  
  

### 서버와 다른 주소에서 javascript 클라이언트 접속
* javascript 클라이언트가 서버와 다른 주소에서 접속할 때는 기본적으로 보안상 금지이다.
* 이것을 막기위해 CORS 규약이 있다. 
* 서버  
```
class Startup
{
	public void Configuration(Owin.IAppBuilder app)
	{
		app.Map("/signalr", map =>
		{
			// Setup the CORS middleware to run before SignalR.
			map.UseCors(CorsOptions.AllowAll);

			var hubConfiguration = new Microsoft.AspNet.SignalR.HubConfiguration
			{
			};

			map.RunSignalR(hubConfiguration);
		});

	}
}
```  
  
* 클라이언트  
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<title></title>
	<script type="text/javascript" src="Scripts/jquery-2.1.1.js"></script>
	<script type="text/javascript" src="Scripts/jquery.signalR-2.0.3.js"></script>
	<script src="http://100.21.6.21:29090/signalr/hubs"></script>
	<script type="text/javascript">
		$(function () {
			$.connection.hub.url = "http://100.21.6.21:29090/signalr";
			$.connection.hub.start(function() {
				alert('connection started!');
			});
		});
	</script>
</head>
<body>

</body>
</html>
```  
  

* 참고 https://sakapon.wordpress.com/tag/asp-net-signalr/