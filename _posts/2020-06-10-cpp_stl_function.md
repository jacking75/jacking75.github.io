---
layout: post
title: C++ - std::function
published: true
categories: [C++]
tags: c++ stl function
---
C++에 존재하는 여러 종류의 함수를 wrap 하여 똑같이 다룬다  
  
C++에 있는 함수의 종류  
- 함수
- 람다식
- 함수 객체  
  
  
  
## 클래스의 멤버 함수
람다식 사용법  
```
  [캡처](인수)->반환 값 타입 { 함수의 내용 }(실행 시 인수);
```
  
- 캡처에는 기본적으로`[=]` 나 `[&]`가 들어간다.
- `=`는 범위 밖의 변수를 이용할 때 복사한다는 의미.
- `&`은 범위 밖의 변수를 이용할 때 참조한다는 의미.
- `[]{}()` 처럼 인수나 반환 값을 생략할 수도 있다.  
  
<br>  
  
```
std::function< 반환 값(인수 타입) > object = 함수 or 
                                             람다식 or 
                                             함수 오브젝트 or 
                                             클래스 멤버 함수;

object(인수); 
```  
로 이용할 수 있다.  
  
<br>  
  
```
#include <iostream>
#include <functional>
using namespace std;

struct Foo {
        Foo(const int n) : i_(n) {}
        void print_add(const int n) const { std::cout << i_ + n<< std::endl; }
        int i_;
};

struct PrintFunctor {
        // 인수 표시만 하는 함수 오브젝트
        void operator()(int i) {
                std::cout << i << std::endl;
        }
};

void print_number(const int i)
{
    std::cout << i << std::endl;
}

int
main(int argc, char const* argv[])
{
  // 보통의 함수
  std::function< void(int) > f_func = print_number;
  f_func(3);

  // 람다 식
  std::function< void(int) > f_lambda = [=](int i) { print_number(i); };
  f_lambda(6);

  // 바인드
  std::function< void() > f_bind = std::bind(print_number, 9);
  f_bind();

  // 클래스 멤버 함수
  std::function< void(const Foo&, int) > f_member = &Foo::print_add;
  Foo foo (1);
  f_member(foo, 3);       // 1+3 = 4

  // 함수 오브젝트
  std::function< void(int) > f_func_obj = PrintFunctor();
  f_func_obj(11);

  return 0;
}
```  
  
  
## 클래스의 멤버 함수를 대입 
클래스  
```
class Main
{
public:
  Main() = default;
  ~Main() = default;

  int Init(ChatServerConfig serverConfig);
  
  void Run();
  
  void Stop();
  
  NetLib::IOCPServerNet* GetIOCPServer() { return  m_pIOCPServer.get(); }


private:
  std::unique_ptr<NetLib::IOCPServerNet> m_pIOCPServer;
  std::unique_ptr<UserManager> m_pUserManager;
  std::unique_ptr<PacketManager> m_pPacketManager;
  std::unique_ptr<RoomManager> m_pRoomManager;
      
  ChatServerConfig m_Config;
      
  bool m_IsRun = false;
};
```  
  
```
int Main::Init(ChatServerConfig serverConfig) //Config를 외부에서 받도록 이 함수의 인
{
  m_pIOCPServer = std::make_unique<NetLib::IOCPServerNet>();
  
  m_pPacketManager = std::make_unique<PacketManager> ();
  m_pUserManager = std::make_unique<UserManager> ();
  m_pRoomManager = std::make_unique<RoomManager> ();

  auto sendPacketFunc = [&](INT32 connectionIndex, void* pSendPacket, INT16 packetSize)
  {
    m_pIOCPServer->SendPacket(connectionIndex, pSendPacket, packetSize);
  };

  m_pPacketManager->SendPacketFunc = sendPacketFunc;
  m_pPacketManager->Init(m_pUserManager.get(), m_pRoomManager.get());
  
  return 0;
}
```  
  
```
class PacketManager {
public:
  PacketManager() = default;
  ~PacketManager() = default;

  std::function<void(INT32, void*, INT16)> SendPacketFunc;
};
```  