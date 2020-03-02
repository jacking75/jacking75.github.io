---
layout: post
title: StackExchange.Redis - score 사용하기
published: true
categories: [.NET]
tags: c# .net StackExchange.Redis redis score
---
StackExchange.Redis 사용 방법 정리.  
  
## 접속 실패 
[출처](https://qiita.com/githiroshi/items/9e26c76fd93bf891fcec )  
```
//Redis의 DB를 선택
IDatabase db = RedisRepository.SelectCache(0);

//key 생성
var key = "hoge";

// score 계산을 위해 Unixtime 얻기
var unixSeconds = DateTimeOffset.Now.ToUnixTimeSeconds();
//경과 초 수를 score로 저장
var score = (double)unixSeconds;

//관람 이력 정보를 정렬된 set 타입으로 등록
db.SortedSetAdd(key, "사과", score);

//보존 기간을 설정하는 경우는 아래처럼(예：30일）
db.KeyExpire(key, new TimeSpan(30, 0, 0, 0));

//최대 건수를 설정하는 경우는 아래처럼
var current = db.SortedSetRangeByScore(key);
var maxCnt = 10;
if (current.Count() > maxCnt)
{
	//최하위의 데이터를 삭제
	db.SortedSetRemoveRangeByRank(key,0,0);
}



//최신 관람 순(scoer 높은 순서)으로 얻는다
var list = db.SortedSetRangeByScore(key, order: Order.Descending);
```
