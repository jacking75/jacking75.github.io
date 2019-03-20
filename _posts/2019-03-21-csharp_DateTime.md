---
layout: post
title: C# - XmlTextReader
published: true
categories: [.NET]
tags: c# .net DateTime
---
## MSDN의 날짜 포맷
- https://msdn.microsoft.com/en-us/library/az4se3k1.aspx
  
  
  
## 현재 시간 중에서 시간:분(09:01)으로 얻고 싶을 때
  
```
DateTime.Now.ToString("hh:mm") // for non military time
DateTime.Now.ToString("HH:mm") // for military time (24 hour clock)
```
  
  
  
## C#에서 String 형식으로 넘어온 날짜와 시간 데이터를 Datetime 형식으로 형변환 하기
  
```
// 출처:http://blog.naver.com/doghole/100117144255
string sDate = “20100127″; string sTime = “16:19″;
// -> DateTime _sdt = DateTime.ParseExact(sDate + ” ” + sTime, “yyyyMMdd H:mm”, null);

string date_string = "201308011121"; // 2013.08.01 11:21
DateTime date = DateTime.ParseExact(date_string, "yyyyMMddHHmm", null);
```
  
  
  
## 두 날짜 사이의 시간 간격 구하기
  
```
// 출처:http://blog.naver.com/doghole/100117144255
TimeSpan tDiff = _edt.Subtract(_sdt);
if (tDiff.TotalHours > 1)
   Console.Write(”두 날짜 사이의 시간 간격이 1시간을 넘어 갑니다.”);
```
  
  
  
## 특정일에서 특정일 더하거나 빼거나 해서 날짜 구하는 법
  
```
// 출처:http://blog.naver.com/doghole/100117144255
// 오늘을 기준으로
// 30일 더하는 방법 :
DateTime.Today.AddDays(30).ToString("yyyyMMdd");
//30일 빼는 방법 :
DateTime.Today.AddDays(-30).ToString("yyyyMMdd");
```
  
  
  
## 시간 포맷
  
```
// 20130530094611 ( 2013년 5월30일 오전 9시 46분 11초)
string reqData = DateTime.Now.ToString("yyyyMMddHHmss");
```
  
다른 포맷은 여기 참조 http://wizcody.egloos.com/2471287
  
  
  
## 현재까지의 tick 시간
- DateTime.Now.Ticks
  
  
  
## tick 단위
- 10000 tick == 1 밀리세컨드
- 10000000 tick = 1 초
  
  
  
## 현재까지의 시간, 분
- TimeSpan을 사용하면 된다.
  
```
var curTimeSec = new TimeSpan(DateTime.Now.Ticks).TotalSeconds;
var curTimeMinu = new TimeSpan(DateTime.Now.Ticks).TotalMiniuts;
```
  
  
  
## 지정한 시간을 지났는지 조사
  
```
public bool CheckTime(Int64 checkTick, Int64 curTick, Int64 timeTick)
{
    var spanTick = curTick - checkTick;

    if (spanTick >= timeTick)
    {
        return true;
    }

    return false;
}
``` 
  
  
  
## (기간)날짜를 tick으로 변환
  
```
public Int64 DayToTick(int day)
{
    var tick = new TimeSpan(day, 0, 0, 0).Ticks;
    return tick
}
``` 
  
  
  
## Tick을 초, 나노 초로 변환
  
```
const Int64 TicksPerMicrosecond = 10;
Int64 sec = (Int64)(curTimeTick / TimeSpan.TicksPerSecond);
Int64 nanoSec = (Int64)(curTimeTick / TicksPerMicrosecond);
```
  
  
  
## FILETIME to DateTime
  
```
long hFTa = (((long)a.dwHighDateTime) << 32) + b.dwLowDateTime;
var a_datetime = DateTime.FromFileTime(hFTa);
```
  
  
  
## 참고
- [(일어)DateTime과 DateTimeOffset의 차이점은?](http://www.atmarkit.co.jp/ait/articles/1506/24/news020.html)
