---
layout: post
title: C++ - localtime_s 사용 예
published: true
categories: [C++]
tags: c++ localtime_s
---
최신 VC 에서는 `localtime`을 사용하면 경고(컴파일 실패 되는)가 나온다.  
`localtime_s`를 사용해야 한다.   
  
```
include<stdio.h>
include<time.h>

int main(void)
{
        time_t curTime = time(NULL);
        struct tm tmCurTime;
        errno_t error;

        error = localtime_s(&tmCurTime, &curTime);
        if (error != 0) 
        {
            printf("현재 시간을 얻을 수 없다.\n");
        }
 }
```  
