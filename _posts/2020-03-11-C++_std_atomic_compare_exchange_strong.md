---
layout: post
title: C++11 - std::atomic::compare_exchange_strong
published: true
categories: [C++]
tags: c++ compare_exchange_strong atomic c++11
---
[출처](https://cpprefjp.github.io/reference/atomic/atomic/compare_exchange_strong.html )  
  
```
bool compare_exchange_strong(T& expected,
                             T desired,
                             memory_order success,
                             memory_order failure
                             ) volatile noexcept;  // (1)
bool compare_exchange_strong(T& expected,
                             T desired,
                             memory_order success,
                             memory_order failure
                             ) noexcept;           // (2)

bool compare_exchange_strong(T& expected,
                             T desired,
                             memory_order order = memory_order_seq_cst
                             ) volatile noexcept;  // (3)
bool compare_exchange_strong(T& expected,
                             T desired,
                             memory_order order = memory_order_seq_cst
                             ) noexcept;           // (4)

```
  
  
  
## 요약
강한 비교로 값을 바꿔 넣는다.
- (1), (2) : 현재 값과 `expected`가 등치 일 경우 `success` 메모리 오더로 현재 값을 `desired`로 교체, 그렇지 않으면 `failure` 메모리 오더로 `expected`를 현재 값으로 대체
- (3), (4) : 현재 값과 `expected`가 등치 일 경우 현재 값을 `desired`로 교체, 그렇지 않으면 `expected`를 현재의 값으로 바꾼다. 양쪽 모두 `order` 메모리 오더가 사용된다
  
  
  
## 요구 사항
`failure`가 `memory_order_release`, `memory_order_acq_rel` 가 아니다  
  
  
  
## 효과
현재 값과 `expected`를 바이트 수준에서 등치 비교를 하고, true이면 현재 값을 `desired`로 교체, false이면 `expected`를 현재 값으로 대체한다.
- (1), (2) : 바이트 등치 비교가 true인 경우 `success` 메모리 오더, false인 경우는 `failure` 메모리 오더에 따라 atomic 값의 대체가 이루어진다
- (3), (4) : atomic 값 대체는 `order` 메모리 오더가 사용된다
  
  
  
## 반환 값
이 함수를 호출하기 전에 `*this`가 보유하는 값과 `expected`의 등치 비교 결과가 반환된다. 등치인 경우 true, 그렇지 않으면 false가 반환된다.  
  
  
  
## 예외
던지지 않는다  
  
  
  
## 비고
이 함수는 값을 바꿀 수 있는 경우 CAS(compare-and-swap) 작업이 항상 성공한다.  
  
`compare_exchange_weak()`는 더 약한 명령이며, 교체 가능한 경우라도 CAS 작업이 실패 할 가능성이 있다.  
  
일반적으로 CAS 작업은 CAS가 성공할 때까지 반복한다.  
  
하지만 CAS 작업에서 Spurious Failure가 발생하지 않으면 반복 할 필요가 없어지는 상황이면 `compare_exchange_strong()`을 사용함으로써 효율적으로 CAS를 할 수 있다.  
  
반대로 말하면, 그런 상황이 아니라면 보통 루프에서 `compare_exchange_weak()`를 이용하면 된다.  
  
  
  
## 예
  
### 기본적인 사용법   
  
```
#include <iostream>
#include <atomic>

int main()
{
  {
    std::atomic<int> x(3);

    // x == expected 이므로 x는 2로 교체 된다
    int expected = 3;
    bool result = x.compare_exchange_strong(expected, 2);

    std::cout << std::boolalpha << result << " " << x.load() << " " << expected << std::endl;
  }
  {
    std::atomic<int> x(3);

    // x != expected 이므로 expected가 x의 값으로 교체된다
    int expected = 1;
    bool result = x.compare_exchange_strong(expected, 2);

    std::cout << std::boolalpha << result << " " << x.load() << " " << expected << std::endl;
  }
}
```
  
결과
<pre>
true 2 3
false 3 3
</pre>
  
  
### 플래그 on/off를 하는 예
  
```
#include <iostream>
#include <atomic>
#include <thread>

class my_atomic_flag {
  std::atomic<bool> value_{false};
public:
  void set() {
    // 값이 false 라면 true로 한다.
    // false가 아니라면 (true), 이 대로
    bool expected = false;
    value_.compare_exchange_strong(expected, true);
  }

  bool load() const {
    return value_.load();
  }
};

int main()
{
  my_atomic_flag x;

  // 어느쪽인가의 스레드의 처리가 끝났다면(성공 하였다면) 플래그를 on 한다
  std::thread t1 {[&x] {
    x.set();
  }};
  std::thread t2 {[&x] {
    x.set();
  }};

  t1.join();
  t2.join();

  std::cout << std::boolalpha << x.load() << std::endl;
}
```
  
결과
<pre>
true
</pre>
  
  
  