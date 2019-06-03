---
layout: post
title: C++ - localtime에서 초 단위의 시간으로 변환 (Code)
published: true
categories: [C++]
tags: c++ time localtime code
---
  
```
struct tm timeinfo;

memset( &timeinfo, 0, sizeof(timeinfo) );
timeinfo.tm_year = atoi( szYear ) - 1900;
timeinfo.tm_mon = atoi( szMonth ) - 1;
timeinfo.tm_mday = atoi( szDay );

DWORD nSecondTime = static_cast<DWORD>(mktime( &timeinfo )); 
```
  