---
layout: post
title: C++ - TimingWheel 알고리즘을 사용한 타이머
published: true
categories: [C++]
tags: c++ timingWheel
---
- 자세한 설명은 http://d2.naver.com/helloworld/267396.  
- 타임아웃 작업의 만료 여부에 대한 검사가 필요 없는 등록과 실행 두 작업만을 모두 O(1)의 시간 복잡도로 처리할 수 있는 타이머를 구현할 수 있다.  
    
출처: https://github.com/billlin0904/librapid  
```C++
uint64_t TimingWheel::add(uint32_t timeout, bool onece, TimeoutCallback callback) {
  //std::lock_guard<platform::Spinlock> guard{ lock_ };
    
  auto bucket = (nextIndex_ + timeout / tickDuration_) % buckets_.size();
  auto id = uint32_t(buckets_[bucket].size());
  
  buckets_[bucket].push_back(std::make_pair(onece, callback));
  
  Slot s;
  s.value[0] = bucket;
  s.value[1] = id;
  return s.pad;
}
```

```C++
void TimingWheel::onTick() {
  //std::lock_guard<platform::Spinlock> guard{ lock_ };

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
  
<br>  
    
전체 코드  
TimingWheel.h  

```C++
class Timer;

class TimingWheel;
using TimingWheelPtr = std::shared_ptr<TimingWheel>;

class TimingWheel {
public:
    using TimeoutCallback = std::function<void(void)>;

  static TimingWheelPtr createTimingWheel(uint32_t tickDuration, uint32_t maxBuckets);

  TimingWheel(uint32_t tickDuration, uint32_t maxBuckets);

    ~TimingWheel();

    TimingWheel(TimingWheel const &) = delete;
    TimingWheel& operator=(TimingWheel const &) = delete;

    void start();

  uint64_t add(uint32_t timeout, bool onece, TimeoutCallback callback);

    void remove(uint64_t slot);

private:
    void onTick();

    union Slot {
        uint64_t pad;
        uint32_t value[2];
    };

    using Bucket = std::vector<std::pair<bool, TimeoutCallback>>;

  details::Timer timer_;
    std::vector<Bucket> buckets_;
    mutable platform::Spinlock lock_;
    uint64_t maxTimeout_;
  uint32_t tickDuration_;
    uint32_t nextIndex_;
};
```    

TimingWheel.cpp  

```C++
static void defaultTimeoutCallback() {
}

TimingWheelPtr TimingWheel::createTimingWheel(uint32_t tickDuration, uint32_t maxBuckets) {
  return std::make_shared<TimingWheel>(tickDuration, maxBuckets);
}

TimingWheel::TimingWheel(uint32_t tickDuration, uint32_t maxBuckets)
    : buckets_(maxBuckets)
  , tickDuration_(tickDuration)
    , nextIndex_(0) {
  maxTimeout_ = tickDuration_ * maxBuckets;
}

TimingWheel::~TimingWheel() {
  std::lock_guard<platform::Spinlock> guard{ lock_ };
    timer_.stop();
}

void TimingWheel::start() {
  timer_.start(std::bind(&TimingWheel::onTick, this), tickDuration_);
}

uint64_t TimingWheel::add(uint32_t timeout, bool onece, TimeoutCallback callback) {
  std::lock_guard<platform::Spinlock> guard{ lock_ };
    
  RAPID_ENSURE(timeout <= maxTimeout_);

  auto bucket = (nextIndex_ + timeout / tickDuration_) % buckets_.size();
  auto id = uint32_t(buckets_[bucket].size());
    
  buckets_[bucket].push_back(std::make_pair(onece, callback));
    
  Slot s;
    s.value[0] = bucket;
    s.value[1] = id;
    return s.pad;
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

void TimingWheel::remove(uint64_t slot) {
  std::lock_guard<platform::Spinlock> guard{ lock_ };

    Slot s;
    s.pad = slot;
    auto bucket = s.value[0];
    auto &chain = buckets_[bucket];
    
  RAPID_ENSURE(s.value[1] < chain.size());
    chain[s.value[1]].second = nullptr;
}
```
  
<br>  
    
사용 예  

```C++
void Connection::addReuseTimingWheel() {
  auto pConn = shared_from_this();

  pTimeWaitReuseTimer_->add(platform::TcpIpParameters::getInstance().getTcpTimedWaitDelay(), true, [pConn]() {
    RAPID_LOG_INFO() << "Background reuse socket";
    if (!pConn->hasUpdataAcceptContext_) { // ?Accept퀂퐑ㄳ?톝㏓뻤acceptAsync
      return;
    }
    try {
      pConn->acceptAsync();
    } catch (Exception const &e) {
      RAPID_LOG_FATAL() << e.what();
    }
  });
}
```