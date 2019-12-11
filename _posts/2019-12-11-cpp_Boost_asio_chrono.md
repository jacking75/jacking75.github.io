---
layout: post
title: C++ - Boost.Asio에서 Boost.Chrono 베이스의 타이머 사용
published: true
categories: [C++]
tags: c++ boost asio boost.chrono
---
Boost 1.49.0 버전에서 Boost.Chrono 베이스의 타이머가 도입 되었다.  
- basic_waitable_timer ： 클락 전환용
- system_timer ： system_clock용 타이머
- steady_timer ： steady_clock용 타이머
- high_resolution_timer ： high_resolution_clock용 타이머
  
  
예제 코드  
```
#include <iostream>
#include <boost/asio.hpp>
#include <boost/asio/steady_timer.hpp>

namespace asio = boost::asio;
namespace chrono = boost::chrono;

void on_timer(const boost::system::error_code& ec)
{
    if (!ec) {
        std::cout << "on timer" << std::endl;
    }
}

int main()
{
    asio::io_service io_service;
    asio::steady_timer timer(io_service); // chrono 베이스의 타이머

    timer.expires_from_now(chrono::milliseconds(100)); 
    timer.async_wait(on_timer);

    io_service.run();
}
```
  