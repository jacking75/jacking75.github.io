---
layout: post
title: C++ - 헤더 파일로 만드는 actor 모델 라이브러리
published: true
categories: [C++]
tags: c++ actor headeronly
---
[원문](http://qiita.com/ToshiManaPlus1/items/fcc10e3e059f7a4e7a95)  
  
- C++의 표준 라이브러리로만 구현(thread, chrono, mutex, atomic, queue)
- 성능보다 구성 요소의 작음을 우선
- [소스](https://github.com/toshimana/MiniActor)
  
  
## actor 모델?
actor 모델은 병렬 계산의 수학적 모델의 일종이다.   
  
병렬 처리 지향이라는 Erlang과 함수형 언어 Scala에서도 사용되고 있다.  
- 어떤 actor 객체는 동시에 하나의(스레드, 프로세스, PC)상에서 동작한다.
- actor들은 메시지에 의한 비동기 통신을 실시한다.
- actor는 개별적으로 메시지 박스를 갖고 있다.
- 액터는 메시지의 송수신으로만 다른 actor와 데이터 교환을 실시한다.
  
  
  
## actor 모델은 무엇이 좋을까?
- 액터의 받은 메시지에 대한 처리는 순차적으로 기재할 수 있다.(메시지 박스에 대한 데이터 입출력부 등은 동기 처리가 필요하지만 쉽게 은폐할 수 있다)
- 동기 처리에 의한 불필요한 대기 시간을 줄일 수 있다.
- lock을 거의 쓰지 않으므로 교착 상태 걱정이 없다.
- 메시지 이외의 입력이 없어서 처리를 파악하기 쉽다.
  
  
  
## MiniActor
actor 모델에 기초한 프로그램을 실현하기 위해서 작성한 싱글 헤더 라이브러리이다.  
- VC2015, GCC4.8.1, clang3.4에서 동작 확인.  
  
  
### 할 수 있는 것
- C++ 표준 라이브러리만 사용해서 다른 외부 라이브러리를 이용하지 않는다
- 스레드 단위의 actor 동작
- 임의의 타입에 의한 메시지 송수신(Template 인수에 의한 메시지 타입 지정)
- 스레드를 멈추지 않는 메세지 송수신(GUI 스레드 등에서 이용)
- Two-Phase Termination(스레드 중단 → 객체 파기)
  
  
### 할수 없는 것
- 고속 처리(성능보다 읽기 쉬움을 중시)
- 많은 actor를 만드는 것(OS의 쓰레드 수에 의존)
- 고도의 예외 처리(예외에 대해서는 거의 아무런 고려가 없다)
  
  
### 사용법
MiniActor의 사용법의 예이다.  
  
  
#### Example01:문자열을 보내기
문자열을 받고 표준 출력으로 출력하는 actor(EchoActor)의 예이다.  
- MiniActor::Actor를 계승한 액터 클래스를 정의한다. 이 경우 템플릿 매개 변수에는 메시지의 타입을 지정한다.
- 받은 메시지의 처리 내용은 process_message 함수에 정의한다.
- actor에 메시지를 보내려면 send 함수를 이용한다.
- actor를 정지할 때는 halt 함수를 사용한다. actor는 종료 상태로 되고, 메시지 처리를 종료하고 자신의 스레드를 정지한다.
  
```
include <iostream>
#include <string>

#include <MiniActor.hpp>

struct EchoActor : public MiniActor::Actor<std::string>
{
    void process_message(const Message& msg)
    {
        std::cout << "received message: " << msg << std::endl;
    }
};

int main()
{
    EchoActor actor;
    actor.send("Test");
    actor.send("Hello");
    actor.send("World");

    std::this_thread::sleep_for(std::chrono::seconds(1));

    actor.halt();

    return 0;
}
```
  
  
#### Example02:actor끼리 통신
2개의 actor가 각각 받은 값을 증가하고 상대측에 돌려주는 예이다.    
  
```
#include <iostream>
#include <string>

#include <MiniActor.hpp>

struct CountActor;
using SpCountActor = std::shared_ptr<CountActor>;
using WpCountActor = std::weak_ptr<CountActor>;

struct CountActor : public MiniActor::Actor<std::pair<int,WpCountActor> >, std::enable_shared_from_this<CountActor>
{
    void process_message(const Message& msg)
    {
        std::cout << std::this_thread::get_id() << " : " << msg.first << std::endl;

        SpCountActor sp = msg.second.lock();
        if (sp) {
            sp->send(std::make_pair(1 + msg.first, shared_from_this()));
        }
    }
};

int main()
{
    SpCountActor actor1 = std::make_shared<CountActor>();
    SpCountActor actor2 = std::make_shared<CountActor>();

    actor1->send( std::make_pair(0, actor2));

    std::this_thread::sleep_for(std::chrono::seconds(1));

    actor1->halt();
    actor2->halt();

    return 0;
}
```
