---
layout: post
title: StackExchange.Redis - list
published: true
categories: [.NET]
tags: c# .net StackExchange.Redis redis
---
StackExchange.Redis 사용 방법 정리.  
  
## 리스트 선두에 추가
[출처](https://kagasu.hatenablog.com/entry/2017/08/29/130804 )  
```
var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379");
var db = redis.GetDatabase();

db.ListLeftPush("mykey", "value1");
db.ListLeftPush("mykey", "value2");
db.ListLeftPush("mykey", "value3");
```
  
  
## 리스트 꼬리에 추가 
[출처](https://kagasu.hatenablog.com/entry/2017/08/29/130804 )   
```
var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379");
var db = redis.GetDatabase();

db.ListRightPush("mykey", "value1");
db.ListRightPush("mykey", "value2");
db.ListRightPush("mykey", "value3");
```
  
  
## 리스트의 요소 수 얻기
[출처](https://kagasu.hatenablog.com/entry/2017/08/29/130804 )   
```
var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379");
var db = redis.GetDatabase();

db.ListLength("mykey")
```