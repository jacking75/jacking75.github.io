---
layout: post
title: Http 호출을 배타적으로 하기
published: true
categories: [.NET]
tags: c# .net http
---
특정 http 호출을 동시에 호출하지 않도록 하는 방법  
[출처](https://qiita.com/ryotkn/items/359e90450bfae332beb5)  
    
```
public static void ExecuteSync(Action SynchronousAction, Action OnBlocked)
{
	string appKey = "MyState";

	var app = System.Web.HttpContext.Current.Application;
	app.Lock();
	var state = app[appKey];
	if (state == null)
	{
		app[appKey] = new object();
	}
	app.UnLock();

	if (state != null)
	{
		OnBlocked();
		return;
	}

	try
	{
		SynchronousAction();
	}
	finally
	{
		app.Lock();
		app.Remove(appKey);
		app.UnLock();
	}
	return;
}
```
   