---
layout: post
title: C# - XmlTextReader
published: true
categories: [.NET]
tags: c# net XmlTextReader xml
---
## XmlTextReader 로 XML 읽기
- 예제 1

<pre>
<?xml version="1.0" encoding="utf-8" ?>
<Config>
  <log4net>
	  <root>
		<!--로그 레벨 지정. 아래에서는 INFO 레벨 이상만 로그가 남는다-->
		<level value="INFO" />
		<!--어느 로그를 사용할 것인지 지정한다. 파일 로그와 날짜별 로그 사용-->
		<appender-ref ref="DayRollingLogToFile" />
		<!--복수 지정 가능-->
	  </root>
  </log4net>

  <UniqueNumber DBConnect="Server=172.20.60.216;Database=Mobile_LogDB;Uid=MobileUser;Pwd=+ahqk@lf!" MaxStockCount="200" MinStockCount="100" />
</Config>
</pre>
  
- 위 xml 파일에서 <UniqueNumber> 태그의 값 읽기
  
```
var reader = new System.Xml.XmlTextReader(xmlFilePath);

reader.ReadToFollowing("UniqueNumber");

LogConfig.DBConnectString = reader.GetAttribute("DBConnect");
LogConfig.MaxStockCount = Convert.ToInt32(reader.GetAttribute("MaxStockCount"));
LogConfig.MinStockCount = Convert.ToInt32(reader.GetAttribute("MinStockCount"));
reader.Close();
```
  
- 예제 2
  
<pre>
<?xml version="1.0" encoding="utf-8" ?>
<Config>
  <DB>
	<MongoDB>
	  <DBINFO DATABASE="GameInfo" COLLECTON="Version" IP="172.20.60.208" />
	  <DBINFO DATABASE="Account" COLLECTON="Login" IP="172.20.60.208" />
	  <DBINFO DATABASE="" COLLECTON="" IP="172.20.60.208" />
	</MongoDB>

	<REDIS>
	  <DBINFO IP="172.20.60.208" PORT="6379" />
	  <DBINFO IP="172.20.60.208" PORT="6380" />
	</REDIS>
  </DB>

  <UniqueNumber URI="http://172.20.60.207:8732" API="CommonLogService/UniqueNumberRange" MaxStockCount="200" MinStockCount="100" />
</Config>
</pre>
   
```   
// <REDIS>의 <DBINFO> 읽기
var reader = new System.Xml.XmlTextReader(logFilePath);

reader.ReadToFollowing("REDIS");
while (reader.ReadToFollowing("DBINFO"))
{
	var ip = reader.GetAttribute("IP");
	var port = Convert.ToInt32(reader.GetAttribute("PORT"));

	RedisAddress.Add(new Tuple<string, int>(ip, port));
}
```
  


