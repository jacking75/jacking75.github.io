---
layout: post
title: C# - ConcurrentQueue, ConcurrentDictionary
published: true
categories: [.NET]
tags: c# .net ConcurrentCollection thread ConcurrentQueue ConcurrentDictionary
---
## ConcurrentQueue
- 자료구조 Queue의 멀티 스레드용 컨테이너
- MSDN http://msdn.microsoft.com/ko-kr/library/dd267265.aspx
  
  
### 정의
  
```
var queue = new System.Collections.Concurrent.ConcurrentQueue<string>();
```
  
  
### 데이터 추가
  
```
queue.Enqueue("First");
```
  
  
### 데이터 참조
  
```
string item;
if (queue.TryPeek(out item))
{
     Console.WriteLine("First item was " + item);
}
else
{
     Console.WriteLine("Queue was empty.");
}
```
  
  
### 데이터 가져오기(삭제)
  
```
if (queue.TryDequeue(out item))
{
     Console.WriteLine("Dequeued first item " + item);
}
```
  
  
### Enumerator
  
```
var queue = new ConcurrentQueue<string>();

// adding to queue is much the same as before
queue.Enqueue("First");

var iterator = queue.GetEnumerator();

queue.Enqueue("Second");
queue.Enqueue("Third");

// only shows First
while (iterator.MoveNext()
```
  
  
  
## ConcurrentDictionary
  
```
using System.Collections.Concurrent;
static ConcurrentDictionary<string, MemoryDBUserAuth> Cache = new ConcurrentDictionary<string, MemoryDBUserAuth>();
```
  
  
### 값 가져오기
  
```
Cache.TryGetValue(userID, out oldAuthInfo) == false)
```
  
  
### 추가
  
```
Cache.TryAdd(userID, newAuthInfo);
```
  
  
### 갱신
  
```
var oldToken = oldAuthInfo.GameAuthToken;
Cache.TryUpdate(userID, newAuthInfo, oldAuthInfo);
```
  
  
### 삭제
  
```	
MemoryDBUserAuth userInfo;
Cache.TryRemove(userID, out userInfo);
```
  