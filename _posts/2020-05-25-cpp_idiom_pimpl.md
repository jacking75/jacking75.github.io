---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) pimpl
published: true
categories: [C++]
tags: c++ idiom pimpl
---
**Pimpl(Pointer to IMPLementation)은 구현을 숨기거나, 헤더를 바꾸지 않는 것으로 컴파일을 속도를 고속화 할 때 사용한다.**  
  
C++20의 module에서는 필요 없어지겠지만 아직은 필요하다  
  
hoge.h  
```
class Hoge {
    int foo_;
    void baz();
public:
    int bar();
    Hoge() = default;
    ~Hoge() = default;
};
```  
  
hoge.cpp  
```
#include "hoge.h"

void Hoge::baz()
{
    // 처리
}
int Hoge::bar()
{
    // 처리
    baz();
}
```  
  
이 클래스에서 새로운 piyo라는 변수를 추가하고 싶은 경우 private 변수로 하더라도 헤더를 다시 써야 한다.  
헤더를 다시 쓰지 않으면 안 되는 것은 이것을 읽고 있는 소스 파일을 다시 한번 컴파일 해야 한다는 것이다.   
  
Pimpl 이디엄을 사용하면 이런 문제를 해결할 수 있다.  
  
hoge.h  
```
#include <memory>

class Hoge {
    class HogeImpl; //전방 선언만 한다
    std::unique_ptr<HogeImpl> impl_; // 이 클래스에서만 사용하므로 unique_ptr을 사용

public:
    Hoge(); 
    ~Hoge(); // pimpl의 경우 여기에 소멸자를 쓸 수 없다.
    int baz();
};  
```
  
hoge.cpp  
```
#include <memory>
#include "hoge.h“

class Hoge::HogeImpl
{
    int foo_;
    void baz() { // 처리 }
Public:
    int bar() { baz(); // 처리 }
};

Hoge::Hoge() : impl_(std::make_unique<HogeImpl>()) { }

Hoge::~Hoge() = default;

int Hoge::bar()
{
    return impl_>bar();
}
```
  
  
소멸자를 헤더에 쓰지 않는 것은 Hogen의 소멸자에서 unique_ptr이 HogeImpl의 소멸자를 호출하기 바라지만 불완전한 클래스인 HogeImpl의 소멸자를 부를 수 없기 때문이다.  
  
pimpl을 사용하면 baz 멤버 함수의 행동을 바꾸기 위해서 적당한 private 변수를 추가하고 싶다든가 할 때에 헤더 파일을 갱신할 필요가 없다.  
  
다음과 같이 하면 된다.  
```
// 생략
Class Hoge::HogeImpl
{
    int foo_;
    int qux_;
    void baz() { qux_ = 1; //처리}

Public:
    int bar() { baz(); // 처리}
};
// 생략
```
  
  
## pimpl의 약점
- 상속 하기 어렵다 
- 원래의 코드보다 (약간)속도가 떨어질 수 있다.
  