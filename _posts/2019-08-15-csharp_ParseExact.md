---
layout: post
title: C# - DateTime의 ParseExact, TryParseExact 메소를 이용하여 문자열 변환
published: true
categories: [.NET]
tags: c# .net DateTime ParseExact TryParseExact
---
## ParseExact 메서드의 문자열 변환
DateTime 구조체에는 문자열을 변환하는 방법으로 Parse 메서드와 ParseExact 메서드가 준비 되어 있지만, 전자의 Parse 메서드는 문화권 정보를 사용하여 출력된 문자열을 같은 문화권 정보를 사용하여 DateTime 개체로 변환하는 것이다. 자신의 형식을 가진 문자열을 취급하는 경우에는 후자의 ParseExact 메서드를 사용해야 한다.  

ParseExact 메서드는 다음과 같이 첫 번째 매개 변수는 입력 문자열을 두 번째 매개 변수는 입력 문자열의 형식을 사용자 지정 형식 문자열로 지정한다. 메소드의 반환 값은 DateTime 형식이다.  

```
DateTime.ParseExact( "2018/08/24 20:23:06", "yyyy/MM/dd HH:mm:ss", null); // HH는 24시간, hh는 12시간
```  
  
세 번째 매개 변수는 문화권별 서식 정보를 제공하는 IFormatProvider 개체를 지정하지만, 두 번째 매개 변수로 지정한 사용자 지정 형식 문자열에 문화권의 형식 지정자가 포함되어 있지 않은 경우는 null로 해도 된다.  
  
  
  
## 날짜 문자열을 DateTime 개체로 변환하는 샘플 프로그램
ParseExact 메서드를 사용하여 날짜 문자열을 DateTime 개체로 변환하는 간단한 샘플 프로그램을 다음과 같다.  
  
```
using System;

public class DateTimeParse {
  static void Main() {

    string d, f;
    DateTime dt;

    d = "2018/08/24 20:23:06";
    f = "yyyy/MM/dd HH:mm:ss";
    dt = DateTime.ParseExact(d, f, null);

    Console.WriteLine(dt.ToString("F"));
    // 出力：2018年8月24日 20:23:06

    d = "20180824202306";
    f = "yyyyMMddHHmmss";
    dt = DateTime.ParseExact(d, f, null);

    Console.WriteLine(dt.ToString("F"));
    // 出力：2018年8月24日 20:23:06

    d = "2018年08月24日20時23分06秒";
    f = "yyyy年MM月dd日HH時mm分ss秒";
    dt = DateTime.ParseExact(d, f, null);

    Console.WriteLine(dt.ToString("F"));
    // 出力：2018年8月24日 20:23:06
  }
}
```  
  
지정한 형식에 맞지 않는 날짜 문자열을 파라메터로 넘긴 경우에는 예외가 발생한다.  
  
```
string d, f;
DateTime dt;

f = "yyyy년MM월dd일HH시mm분ss초";

// 지정한 형식에 일치 하지 않는 경우
d = "2018년08월24일20시23분6초"; // 초의 앞에 0이 없다
try
{
  dt = DateTime.ParseExact(d, f, null);
}
catch (Exception ex)
{
  Console.WriteLine(string.Format("{0}: {1}", ex.GetType().Name, ex.Message));
  // 출력：FormatException: 문자열은 유효한 DateTime 이 아니다.
}

// 있을 수 없는 시간
d = "2018년02월29일20시23분06초"; // 2018년에는 2/29일이 없다
try
{
  dt = DateTime.ParseExact(d, f, null);
}
catch (Exception ex)
{
  Console.WriteLine(string.Format("{0}: {1}", ex.GetType().Name, ex.Message));
  // 출력：FormatException: 문자열로 표시되는 DateTime가 달력 System.Globalization.GregorianCalendar 에서 지원하지 않습니다.
}
```  
  
  
## 예외를 내지 않는 TryParseExact 메서드 [.NET 2.0 이상]
DateTime 구조체에 .NET Framework 2.0에서 도입된 TryParseExact 메서드는 날짜 문자열을 해석 할 수 없는 경우에 예외를 발생시키지 않고 메소드의 반환 값으로 false를 반환한다. 아래의 코드는 예외 처리의 복잡한 코드를 작성하지 않는다.  
  
```
// 선두에「using System;」과 「using System.Globalization;」가 필요

string d, f;
DateTime dt;

d = "2016/02/29 01:23:45";
f = "yyyy/MM/dd HH:mm:ss";
if (DateTime.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dt))
  Console.WriteLine($"{dt:F} ({dt.Kind})");
// 출력：2016년2월29일 1:23:45 (Local)
// ※ 2016년 윤초

d = "2018/02/29 01:23:45";
if (!DateTime.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dt))
  Console.WriteLine($"\"{d}\"는 날짜로 올바르지 않습니다");
// 출력："2018/02/29 01:23:45"은 날짜로 올바르지 않습니다.

d = "2018-08-24T11:23:06.000Z"; // 꼬리의 「Z」는 UTC（세계 협정시）를 뜻한다
f = "yyyy-MM-ddTHH:mm:ss.fffZ";
if (DateTime.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dt))
  Console.WriteLine($"{dt:F} ({dt.Kind})");
// 출력：2018년8월24일 20:23:06 (Local)

d = "2018-08-24T20:23:06.000+09:00";
f = "yyyy-MM-ddTHH:mm:ss.fffzzz"; //「zzz」는 UTC에서 옵셋을 표시한다
if (DateTime.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dt))
  Console.WriteLine($"{dt:F} ({dt.Kind})");
// 출력：2018년8월24일 20:23:06 (Local)
```  
  
  
  
## UTC 오프셋을 고려한다면 DateTimeOffset 구조체 [.NET 2.0 이상]
여기까지 소개해온 DateTime 구조체는 UTC(협정 세계시)와 오프셋(시차)를 가질 수 없다. 오프셋도 쓸 경우에는 .NET Framework 2.0에서 도입된 DateTimeOffset 구조체(System 네임 스페이스)를 이용한다.  
  
날짜 문자열을 DateTimeOffset 개체로 변환하려면 DateTimeOffset 구조체의 TryParseExact 메서드를 사용한다. 사용법은 DateTime 구조체의 TryParseExact 메서드와 동일하다 (아래 코드).  
  
```
string d, f;
DateTimeOffset dto;

d = "2016/02/29 01:23:45";
f = "yyyy/MM/dd HH:mm:ss";
if (DateTimeOffset.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dto))
  Console.WriteLine($"{dto} (local time={dto.ToLocalTime():F})");
// 출력：2016/02/29 1:23:45 +09:00 (local time=2016년2월29일 1:23:45)

d = "2018/02/29 01:23:45";
if (!DateTimeOffset.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dto))
  Console.WriteLine($"\"{d}\"는 날짜로서 올바르지 않습니다");
// 출력："2018/02/29 01:23:45"는 날짜로서 올바르지 않습니다

d = "2018-08-24T11:23:06.000Z";
f = "yyyy-MM-ddTHH:mm:ss.fffZ";
if (DateTimeOffset.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dto))
  Console.WriteLine($"{dto} (local time={dto.ToLocalTime():F})");
// 출력：2018/08/24 11:23:06 +00:00 (local time=2018년8월24일 20:23:06)

d = "2018-08-24T20:23:06.000+09:00";
f = "yyyy-MM-ddTHH:mm:ss.fffzzz";
if (DateTimeOffset.TryParseExact(d, f, null, DateTimeStyles.AssumeLocal, out dto))
  Console.WriteLine($"{dto} (local time={dto.ToLocalTime():F})");
// 출력：2018/08/24 20:23:06 +09:00 (local time=2018년8월24일 20:23:06)
```  
  
  
  
## ISO 8601 형식이라면 TryParse 메소드로 괜찮다
여기까지 입력되는 날짜 문자열의 형식이 고정이라고 해왔다. 그런데, Web 서비스 등은 그렇지 않다. 클라이언트의 JavaScript를 생성하는 날짜 문자열의 패턴은 하나가 아니기 때문이다. 그러나 Web으로 주고받는 날짜 문자열의 형식은 ISO 8601 형식으로 정해져있다(정확하게는 ISO 8601 부분 집합). 앞의 샘플 코드 끝에 "Z"와 "+9:00"라고 붙여 있는 날짜 문자열이 ISO 8601 형식이다.  
  
날짜 문자열에 많은 패턴이 있으면 이것을 TryParseExact 메서드로 해석하는 것은 힘들다. ISO 8601 형식임을 알고 있다면, TryParse 메소드를 사용하면 된다.  
  
```
string d;
DateTimeOffset dto;

d = "2018-08-24T11:23Z";
if (DateTimeOffset.TryParse(d, out dto))
  Console.WriteLine($"\"{d}\" ⇒ {dto} (local time={dto.ToLocalTime():F})");
// 출력："2018-08-24T11:23Z" ⇒ 2018/08/24 11:23:00 +00:00 (local time=2018년8월24일 20:23:00)

d = "2018-08-24T16:23:06+05:00";
if (DateTimeOffset.TryParse(d, out dto))
  Console.WriteLine($"\"{d}\" ⇒ {dto} (local time={dto.ToLocalTime():F})");
// 출력："2018-08-24T16:23:06+05:00" ⇒ 2018/08/24 16:23:06 +05:00 (local time=2018년8월24일 20:23:06)

d = "2018-08-24T20:23:06.000+09:00";
if (DateTimeOffset.TryParse(d, out dto))
  Console.WriteLine($"\"{d}\" ⇒ {dto} (local time={dto.ToLocalTime():F})");
// 출력："2018-08-24T20:23:06.000+09:00" ⇒ 2018/08/24 20:23:06 +09:00 (local time=2018년8월24일 20:23:06)
```  
  
  
<br>  
  
  
[출처:](http://www.atmarkit.co.jp/ait/articles/0409/03/news087.html )  