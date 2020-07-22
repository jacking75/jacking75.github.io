---
layout: post
title: C++ - malloc으로 확보한 메모리를 unique_ptr로 관리하기
published: true
categories: [C++]
tags: c++ unique_ptr
---
[원문](https://qiita.com/saba-miso/items/167022d02660b6549112   )  
  
호출 예  
hoge.h  
```
#pragma once

struct Hoge
{
    int _data1;
    unsigned char _data2[1];
};
```
  
main.cc  
```
#include "uniqueptr1.h"
#include "uniqueptr2.h"
#include "uniqueptr3.h"

int main()
{
    auto p1 = CreateHoge1(100);
    auto p2 = CreateHoge2(100);
    auto p3 = CreateHoge3(100);
    return 0;
}
```
  
  
## 커스텀 딜러터에 함수 오브젝트를 사용하는 경우
free 함수를 호출하기만을 위한 목적으로 class(또는 struct)를 필요로 하는 것은 번잡스럽다.  
  
uniqueptr1.h  
```
#pragma once

#include <memory>
#include "hoge.h"

class MyCustomDeleter
{
public:
    void operator()(void* p) const;
};

std::unique_ptr<Hoge, MyCustomDeleter> CreateHoge1(int dataSize);
```
  
uniqueptr1.cc  
```
#include "uniqueptr1.h"
#include <stdlib.h>

std::unique_ptr<Hoge, MyCustomDeleter> CreateHoge1(int dataSize)
{
    auto hoge = static_cast<Hoge*>(malloc(sizeof(Hoge) - 1 + dataSize));
    return std::unique_ptr<Hoge, MyCustomDeleter>(hoge, MyCustomDeleter());
}

void MyCustomDeleter::operator()(void* p) const
{
    free(p);
}
```
  
  
## 커스텀 딜리터에 std::function을 사용하는 경우
uniqueptr2.h  
```
#pragma once

#include <functional>
#include <memory>
#include "hoge.h"

std::unique_ptr<Hoge, std::function<void(Hoge*)>> CreateHoge2(int dataSize);
```
  
uniqueptr2.cc  
```
#include "uniqueptr2.h"
#include <stdlib.h>

std::unique_ptr<Hoge, std::function<void(Hoge*)>> CreateHoge2(int dataSize)
{
    auto hoge = static_cast<Hoge*>(malloc(sizeof(Hoge) - 1 + dataSize));
    return std::unique_ptr<Hoge, std::function<void(Hoge*)>>(hoge, &free);
}
```
  
  
## 커스텀 딜리터에 함수 포인터를 사용하는 경우
uniqueptr3.h  
```
#pragma once

#include <memory>
#include "hoge.h"

std::unique_ptr<Hoge, void (*)(void* p)> CreateHoge3(int dataSize);
```
  
uniqueptr3.cc  
```
#include "uniqueptr3.h"
#include <stdlib.h>

std::unique_ptr<Hoge, void (*)(void* p)> CreateHoge3(int dataSize)
{
    auto hoge = static_cast<Hoge*>(malloc(sizeof(Hoge) - 1 + dataSize));
    return std::unique_ptr<Hoge, void (*)(void* p)>(hoge, &free);
}
```
  
