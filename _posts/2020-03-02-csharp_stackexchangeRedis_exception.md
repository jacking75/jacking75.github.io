---
layout: post
title: StackExchange.Redis - Json.Net과 같이 사용할 때 예외가 발생하는 경우
published: true
categories: [.NET]
tags: c# .net StackExchange.Redis redis
---
StackExchange.Redis 사용 방법 정리.  
  
## 접속 실패 
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
try
{
    cache = Connection.GetDatabase();
}
catch (RedisConnectionException e)
{
    WriteException("접속 실패", e);
    return;
}
```
  
  
## key가 없는 경우
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
try
{
	// 존재하지 않는 key를 StringGet 메소드에 넘기면 RedisValue 타입에서 Null을 가리키는 값을 반환한다.
	// Null인데 Json 디코딩을 해서 예외 발생
    var notExist = JsonConvert.DeserializeObject<EmployeeDto>(cache.StringGet(key));
}
catch (ArgumentException e)
{
    WriteException("key가 존재하지 않는 경우", e);
}
```
  
  
## String 타입을 저장해야 하는데 다른 타입으로 값을 저장할 때 RedisServerException 발생
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
key = "other-type";
HashEntry[] hashEntries = {
    new HashEntry("a", 1),
};
cache.HashSet(key, hashEntries);
try
{
    var hash = JsonConvert.DeserializeObject<EmployeeDto>(cache.StringGet(key));
}
catch (RedisServerException e)
{
    WriteException("String 타입이 아닌 경우", e);
}
```
  
  
## parse 하면 JSON과 맵핑 대상의 클래스가 일치하지 않는 경우 
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
key = "anonymous";
var objDto = new { T = "a" };
cache.StringSet(key, JsonConvert.SerializeObject(objDto));
var eAnonymous = JsonConvert.DeserializeObject<EmployeeDto>(cache.StringGet(key));
try
{
    Write(key, eAnonymous);
}
catch (NullReferenceException e)
{
    WriteException("parse 하면 JSON과 맵핑 대상의 클래스가 일치하지 않는 경우", e);
}
```
  
  
## 내용이 JSON이 아닌 그냥 문자열인 경우 
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
key = "string";
var stringDto = "test";
cache.StringSet(key, stringDto);
EmployeeDto str = null;
try
{
    str = JsonConvert.DeserializeObject<EmployeeDto>(cache.StringGet(key));
    Write(key, str);
}
catch (JsonReaderException e)
{
    WriteException("내용이 JSON이 아닌 그냥 문자열인 경우", e);
}
```
  
  
## 내용이 배열인 경우
[출처](http://igatea.hatenablog.com/entry/2018/03/23/002937 )  
```
key = "array";
var badJsonDto = "[1, 2, 3]";
cache.StringSet(key, badJsonDto);
try
{
    var arr = JsonConvert.DeserializeObject<EmployeeDto>(cache.StringGet(key));
    Write(key, arr);
}
catch (JsonSerializationException e)
{
    WriteException("내용이 배열인 경우", e);
}  
```