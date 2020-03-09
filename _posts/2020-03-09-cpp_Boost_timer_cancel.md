---
layout: post
title: C++ - Boost 타이머 취소하기
published: true
categories: [C++]
tags: c++ boost boost.timer
---
[출처](https://amedama1x1.hatenablog.com/entry/2016/04/29/200724 )   
  
## 타이머를 다시 이용하지 않는 경우
상태 변수를 추가하지 않고 취소를 알 수 있다.  
```
#include <iostream>
#include <boost/asio.hpp>
#include <boost/asio/steady_timer.hpp>

int main()
{
  using namespace boost::asio;
    
  io_service io_srv{};

  steady_timer timer{io_srv, std::chrono::nanoseconds{1}};
  timer.async_wait([&](boost::system::error_code ec) {
    if (ec || steady_timer::clock_type::now() < timer.expires_at()) {
      std::cout << "timer is cancelled - " << ec << std::endl;
    }
    else {
      std::cout << "timer is not canncelled" << std::endl;
    }
  });

  io_srv.post([&]{
    std::cout << "start work" << std::endl;
    timer.expires_at(steady_timer::time_point::max());
    std::cout << "cancel timer" << std::endl;
  });

  io_srv.run();
}
```
  
멤버 함수 cancel 대신 타임 아웃 시간을 설정하는 expires_at을 사용한다.  
사용하는 타이머 개체가 취할 수 있는 최대 시간을이 함수에 전달한다  .
타임 아웃 핸들러에서의 타임 아웃을 판정하는 조건으로 현재 시간과 타이머 개체의 타임 아웃 시간을 비교하도록 하고 있다.  
  
취소한 경우 타이머 개체는 머나 먼 미래의 시간이 설정 되어 있기 때문에, 현재 시간과 비교하여 취소 되었는지 여부를 확인할 수 있다. 반대로 취소하지 않으면 현재 시간은 타임 아웃 시간보다도 뒤다.  
  
또한 타임 아웃 시간을 설정하는 expires_at는 멤버 함수 cancel과 마찬가지로 타임 아웃 대기 핸들러를 취소하기 때문에 오류 코드가 취소 값이 되는 경우가 있다.  
  
이 방법에서 하나 주의 할 것은, 타이머 개체의 수명이다. 타임 아웃 핸들러에서 타이머 객체를 조작하기 때문에 동적 타이머 객체를 생성하는 경우 이 타이머를 가리키는 shared_ptr을 핸들러에 갖게 할 필요가 있을 것이다.  
  
  
## 타이머를 다시 이용하는 경우
하나의 타이머 객체를 여러 번 재사용 할 경우 각 시간 처리에 대해 ID를 흔들어 주면 좋다.  
```
#include <cstddef>
#include <iostream>
#include <boost/asio.hpp>
#include <boost/asio/steady_timer.hpp>

int main()
{
  using namespace boost::asio;
    
  io_service io_srv{};

  steady_timer timer{io_srv, std::chrono::nanoseconds{1}};
  std::size_t timer_id = 0;
    
  timer.async_wait([&, current_id = timer_id](boost::system::error_code ec) {
    if (ec || current_id != timer_id) {
      std::cout << "timer is cancelled - " << ec  << std::endl;
    }
    else {
      std::cout << "timer is not canncelled" << std::endl;
    }
  });

  io_srv.post([&]{
    std::cout << "start work" << std::endl;
    ++timer_id;
    timer.cancel();
    std::cout << "cancel timer" << std::endl;
  });

  io_srv.run();
}
```
  
타이머 개체마다 타이머 ID를 의미하는 카운터 변수를 정의하고, 타임 아웃 핸들러에 현재 ID를 갖게하고, 타임 아웃 판정 처리는 유지하는 타이머 ID와 현재의 타이머 ID가 일치하는지 확인한다.

취소 처리에서 타이머 ID를 증가 해주면 핸들러가 보관 유지하는 ID와 현재 ID에 차이가 생기므로 타임 아웃을 감지 할 수 있다.  
  