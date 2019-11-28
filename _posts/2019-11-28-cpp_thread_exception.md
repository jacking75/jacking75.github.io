---
layout: post
title: C++ - 스레드 내에서 발생한 예외 처리
published: true
categories: [C++]
tags: c++ c++11 exception_ptr std::thread
---
[참고](https://codezine.jp/article/detail/10580?p=3)  
아래의 코드는 sigma_reference 함수에서 예외가 발생하는 경우 main에서 catch하고 있지만 예외가 잡히지 않는다.  
이유는 sigma_reference 함수의 스레드와 main 함수의 스레드가 서로 다르기 때문이다.  
```
#include <thread>
#include <iostream>
#include <functional> // ref
#include <stdexcept> // invalid_argument
#include <string>    // to_string

/*
 * n < 0 때 invalid_argument 예외를 throw 한다
 */
void sigma_reference(int n, int& result) {
  if ( n < 0 ) {
    throw std::invalid_argument(std::to_string(n));
  }
  int sum = 0;
  for ( int i = 0; i <= n; ++i ) {
    sum += i;
  }
  result = sum;
}

int main() {
  using namespace std;

  int result;
  for ( int n : { 5, -3, 10 } ) try {
    thread thr(sigma_reference, n, ref(result));
    thr.join();
    cout << "1+2+...+" << n <<  "= " << result << endl;
  } catch ( invalid_argument& err ) {
    cerr << err.what() << endl;    
  }

}
```  
  
exception_ptr를 사용하면 위 문제를 해결할 있다.  
```
#include <iostream>
#include <thread>
#include <functional> // ref
#include <stdexcept>  // invalid_argument
#include <exception>  // current_exception/rethrow_exception
#include <string>     // to_string

void sigma_reference(int n, int& result) {
  if ( n < 0 ) {
    throw std::invalid_argument(std::to_string(n));
  }
  int sum = 0;
  for ( int i = 0; i <= n; ++i ) {
    sum += i;
  }
  result = sum;
}

/*
 * sigma_reference 함수를 랩핑한다
 */
void sigma_reference_wrap(int n, int& result, std::exception_ptr& exp) try {
  sigma_reference(n, result);
} catch (...) {
  // catch 된 예외(가 wrap된 exception_ptr)를 set 한다
  exp = std::current_exception();
  // 스레드는 이 시점에서 종료한다
}

int main() {
  using namespace std;

  int result;
  for ( int n : { 5, -3, 10 } ) try {
    exception_ptr exp;
    thread thr(sigma_reference_wrap, n, ref(result), ref(exp));
    thr.join();
	
    // 스레드 내에서 예외고 발생하고 있다면 다시 throw
    if ( exp ) {
      rethrow_exception(exp);
    }
    cout << "1+2+...+" << n <<  "= " << result << endl;
  } catch ( invalid_argument& err ) { // 다시 throw된 에외를 catch
    cerr << err.what() << endl;    
  }

}
```  
  
  
