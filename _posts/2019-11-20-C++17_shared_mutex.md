---
layout: post
title: C++17 - shared_mutex 
published: true
categories: [C++]
tags: c++ c++17 shared_mutex
---
[출처](https://codezine.jp/article/detail/10640?p=3)  
  
mutex로 보호 된 데이터를 다수의 스레드가 액세스 할 때 성능이 저하된다.  
이 데이터 변경을 하지 않는 스레드가 다수인 경우 성능 저하를 줄일 수 있다.  
  
데이터의 변경을 하지 않는 읽기(reader) 스레드와 데이터의 변경을 하는 쓰기(writer) 스레드가 각각 복수 동작하고 있다고 하자. 쓰기는 다른 쓰기/ 읽기가 있을 때는 쓸 수 없다. 그에 비해 읽기는 데이터를 변경하지 않는 읽기만 하는 스레드이기 때문에, 다른 쓰기가 없다면 많은 쓰기 스레드가 읽어도 괜찮다.  
  
보통의 mutex는 일기/쓰는 구별이 없기 때문에 데이터의 변경을 하지 않아도 lock을 획득 할 수 있는 것은 하나의 스레드 만이다.  
C++ 17에 새로 들어간 std::shared_mutex는 쓰기를 위해 lock() / try_lock() / unlock() 뿐만 아니라 읽기를 위한 lock_shared() / try_lock_shared() / unlock_shared()가 준비 되어 있다.  
  
아래 예제 코드.  
writer 스레드는 전역 변수 x, y에 1 ~ 9의 난수, z에 x와 y의 곱을 쓴다.  
reader 스레드는 x * y가 z와 같은지 여부를 확인한다.  
하나의 writer 스레드와 7개의 reader 스레드를 시작하고, 모든 스레드가 완료 될 때까지의 시간을 측정 해 보았다.     
```
#include <thread>
#include <chrono>
#include <random>
#include <iostream>
#include <shared_mutex>

int x;
int y;
int z;

int bad_results;

std::shared_mutex mtx;

void writer() {
  using namespace std;
  mt19937 gen;
  uniform_int_distribution<int> dist(1,9);
  for ( int i = 0; i < 10000000; ++i ) {
    mtx.lock();
    x = dist(gen);
    y = dist(gen);
    z = x * y;
    mtx.unlock();
  }
  z = -1; 
}

// （write-lock 하는）보통의 reader
void reader() {
  while ( true ) {
    mtx.lock();
    if ( z < 0 ) { mtx.unlock(); break; }
    if ( z != x * y ) ++bad_results;
    mtx.unlock();
  }
}

// read-lock 하는 shared_reader
void shared_reader() {
  while ( true ) {
    mtx.lock_shared();
    if ( z < 0 ) { mtx.unlock_shared(); break; }
    if ( z != x * y ) ++bad_results;
    mtx.unlock_shared();
  }
}

void run(bool shared) {
  using namespace std;
  using namespace std::chrono;

  x = 0;
  y = 0;
  z = 0;
  bad_results = 0;

  const int Nread = 7;
  thread threads[Nread+1];

  auto start = chrono::high_resolution_clock::now();
  threads[0] = thread(writer);
  for ( int i = 1; i <= Nread; ++i ) {
    threads[i] = thread(shared ? shared_reader : reader);
  }
  for ( thread& thr : threads ) thr.join();
  auto stop = chrono::high_resolution_clock::now();
  cout << chrono::duration_cast<chrono::milliseconds>(stop - start).count() << "[ms] "
       << "bad_results = " << bad_results << endl;
}

int main() {
  std::cout << "normal: "; run(false);
  std::cout << "shared: "; run(true);
}
```    
      