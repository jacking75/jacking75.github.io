---
layout: post
title: C# - AutoMapper
published: true
categories: [.NET]
tags: c# net AutoMapper
---
## 기본
- 객체간의 복사를 자동으로 한다. 복사되는 객체에 복사한다.
- Nuget으로 설치 가능.
- 예
  
```
public class SourceType
{
    public int SourceId { get; set; }
    public string Value { get; set; }
    public DateTime DateUpdated { get; set; }
}

public class TargetType
{
    public string Value { get; set; }
    public DateTime DateUpdated { get; set; }
}

Mapper.CreateMap<SourceType, TargetType>();

var source = new SourceType() { SourceId = 1, Value = "value", DateUpdated = DateTime.Today };
    var target = Mapper.Map<TargetType>(source);
```
  
  
  
## 객체간의 멤버 이름이 서로 다른 경우
  
```
public class SourceType
{
  public int SourceId { get; set; }
  public string Value { get; set; }
  public DateTime DateUpdated { get; set; }
}
 
public class TargetType
{
  public string Value { get; set; }
  public DateTime DateChanged { get; set; }
}

Mapper.CreateMap<SourceType, TargetType>()
      .ForMember(d => d.DateChanged, o => o.MapFrom(s => s.DateUpdated));

var source = new SourceType() { SourceId = 1, Value = "value", DateUpdated = DateTime.Today };
var target = Mapper.Map<TargetType>(source);
```
  
  
  
## 여러 소스를 하나의 타켓에 매핑 
  
```
public class SourceType1
{
  public int SourceId { get; set; }
  public string Value { get; set; }
  public DateTime DateUpdated { get; set; }
}
 
public class SourceType2
{
  public int Rank { get; set; }
}
 
public class TargetType
{
  public string Value { get; set; }
  public DateTime DateChanged { get; set; }
  public int CurrentRank { get; set; }
}

// SourceType1과 SourceType2를 TargetType으로 매핑
Mapper.CreateMap<SourceType1, TargetType>()
      .ForMember(d => d.DateChanged, o => o.MapFrom(s => s.DateUpdated));
 
Mapper.CreateMap<SourceType2, TargetType>()
      .ForMember(d => d.CurrentRank, o => o.MapFrom(s => s.Rank));

var source1 = new SourceType1() { SourceId = 1, Value = "value", DateUpdated = DateTime.Today };
var source2 = new SourceType2() { Rank = 2 };
 
var target = Mapper.Map<TargetType>(source1);
target = Mapper.Map<SourceType2, TargetType>(source2, target);
```
  
```
// 확장 메소드를 사용하면 좀 더 간단
public static TTarget Map<TSource, TTarget>(this TTarget target, TSource source)
{
  var result = Mapper.Map(source, target);
  return result;
} 

var source1 = new SourceType1() { SourceId = 1, Value = "value", DateUpdated = DateTime.Today };
var source2 = new SourceType2() { Rank = 2 };
 
var target = Mapper.Map<TargetType>(source1)
                   .Map(source2);
```
  
  
  
## 참고 및 출처
- AutoMapper 소개 
    - http://blog.aliencube.org/ko/2015/03/24/introducing-automapper/
- (일어)AutoMapper를 사용하여 서로 다은 오브젝트 간에 데이터 복사를 자동화
    - http://www.atmarkit.co.jp/ait/articles/1503/17/news115.html
    - http://www.atmarkit.co.jp/ait/articles/1503/24/news062.html
	- http://www.atmarkit.co.jp/ait/articles/1504/01/news017.html
