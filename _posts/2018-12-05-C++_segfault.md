---
layout: post
title: C++ - segmentation fault를 보완해주는 로그를 출력하자
published: true
categories: [C++]
tags: c++ segmentation_fault
---
[출처](https://qiita.com/mas-yo/items/2c03538ee3adddaa81c9 )  

C++ 서버(Linux)가 크래쉬 되는 버그로 고생해서 segmentation fault를 보완해주는 로그를 출력하는 기능을 만들어 보았다.  

### 코드 
CrashReporter.hpp  
```
#pragma once

class ICrashReporter
{
public:
    virtual void CrashReport() = 0;
};

class CCrashReporter
{
public:
    static void Setup();
    static void AddReporter(ICrashReporter*);
    static void Start();
};
```  

CrashReporter.cpp  
```
#include "crashreporter.hpp"
#include <signal.h>
#include <unistd.h>
#include <vector>
#include <thread>
#include <csignal>

volatile std::sig_atomic_t _crashed = 0;
std::vector<ICrashReporter*> _reporters;

static void segv_handler(int sig)
{
    _crashed = 1;
}

static void check_crash()
{
    if (_crashed)
    {
        printf("***** CRASH REPORT *****\n");
        for (ICrashReporter* reporter : _reporters)
        {
            reporter->CrashReport();
        }
        printf("************************\n");
        signal(SIGSEGV, SIG_DFL);
        kill(getpid(), SIGSEGV);
    }
}

void CCrashReporter::Setup()
{
    signal(SIGSEGV, segv_handler);
}

void CCrashReporter::AddReporter(ICrashReporter* reporter)
{
    _reporters.push_back(reporter);
}

void CCrashReporter::Start()
{
    std::thread th([]()
    {
        while(true)
        {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            check_crash();
        }
    });

    th.detach();
}
```  
  
  
### 사용 법 

```
class Hoge : public ICrashReporter
{
public:
    virtual void CrashReport() override {
        printf("출력하고 싶은 로그\n");
    }
};

int main()
{
    CCrashReporter::Setup();

    auto hoge = new CHoge();
    CCrashReporter::AddReporter(hoge);
    CCrashReporter::Start();

    //hoge를 사용한 처리...

    return 0;
}
```