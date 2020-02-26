---
layout: post
title: C++ - Win32API 스레드 풀을 사용하는 타이머
published: true
categories: [C++]
tags: c++ boost object_pool
---  
- Windows Vista부터 사용 가능.
- CreateThreadpoolTimer, SetThreadpoolTimer API 사용.
- '윈도우 시스템 프로그램을 구현하는 기술(한빛미디어)'에 설명이 잘 나와 있음  
  
<br>  
    
예제 코드 출처: https://github.com/billlin0904/librapid  
```
using TimerPtr = std::shared_ptr<Timer>;

class Timer {
public:
  using TimeoutCallback = std::function<void(void)>;

  static TimerPtr createTimer();
    
  void start(TimeoutCallback callback, uint32_t interval, bool immediately = true, bool once = false);

  void stop();
private:
  void timeout();

  static void CALLBACK onTimeoutCallback(PTP_CALLBACK_INSTANCE instance, PVOID param, PTP_TIMER timer);

  TimeoutCallback onTimeoutCallback_;
  TP_CALLBACK_ENVIRON callbackEnviron_;
  PTP_TIMER handle_;
};

inline TimerPtr Timer::createTimer() {
  return std::make_shared<Timer>();
}
```
```
Timer::Timer()
  : handle_(nullptr) {
  ::InitializeThreadpoolEnvironment(&callbackEnviron_);
  handle_ = ::CreateThreadpoolTimer(onTimeoutCallback, this, &callbackEnviron_);
  if (!handle_) {
    //throw Exception(::GetLastError());
  }
}

Timer::~Timer() {
  stop();
}

void Timer::start(TimeoutCallback callback, uint32_t interval, bool immediately, bool once) {
  onTimeoutCallback_ = callback;
  FILETIME dueTime;
  *reinterpret_cast<PLONGLONG>(&dueTime) = -static_cast<LONGLONG>(MILLI_SECOND_TO_NANO100(0));
  ::SetThreadpoolTimer(handle_, &dueTime, once ? 0 : interval, 0);
}

void Timer::stop() {
  if (handle_ != nullptr) {
    ::SetThreadpoolTimer(handle_, nullptr, 0, 0);
    ::WaitForThreadpoolTimerCallbacks(handle_, TRUE);
    ::CloseThreadpoolTimer(handle_);
    ::DestroyThreadpoolEnvironment(&callbackEnviron_);
    handle_ = nullptr;
  }
}

void Timer::timeout() {
  try {
    onTimeoutCallback_();
  } catch (...) { }
}
```
  
사용 예  
```
{
  timer_.start(std::bind(&TimingWheel::onTick, this), tickDuration_);
}

class TimingWheel
{
  uint32_t tickDuration_;
}

void TimingWheel::onTick() {
  std::lock_guard<platform::Spinlock> guard{ lock_ };

    auto & bucket = buckets_[nextIndex_];
    for (auto &callback : bucket) {
    if (callback.second != nullptr) {
      callback.second();
      if (callback.first) {
        callback.second = nullptr;
      }
    }
    }

    nextIndex_ = (nextIndex_ + 1) % buckets_.size();
}
```
  
