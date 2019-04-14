---
layout: post
title: C# - ScriptCs.Request
published: true
categories: [.NET]
tags: c# .net ScriptCs
---
## 개요
- http 요청 모듈
- https://github.com/martinobrink/ScriptCs.Request
  
  
  
## 설치
  
```
scriptcs -install ScriptCs.Request
```
  
  
## 사용법
  
```
public class Notification
{
    public string Message {get; set;}
    public string SenderName {get; set;}
}

var request = new Request();

var notifications = request.GetJson<List<Notification>>("http://your.site.com/api/notification");
Console.WriteLine("First notification message: " + notifications[0].Message);

var notificationToSend = new Notification { Message = "Merry christmas!", SenderName = "Santa"};
var response = request.PostJson("http://your.site.com/api/notification", notificationToSend);
Console.WriteLine("Response status code: " + response.StatusCode);
```
  