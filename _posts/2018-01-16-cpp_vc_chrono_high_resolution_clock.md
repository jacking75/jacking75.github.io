---
layout: post
title: C++ - Visual C++ 에서의 std::chrono의 고해상도 시계
published: true
categories: [C++]
tags: c++ c++11 vc 
---
Visual Studio 2015 이전에는 std::chrono(high_resolution_clock을 사용해도)로 시간 측정을 하면 nano 단위로는 측정할 수 없었다.    
그래서 VC에서 nano 단위로 측정하려면 이전처럼 Win32 API인 PerformanceCounter를 사용해야 했다.   
그러나 Visual Studio 2015부터는 nano 단위의 측정이 가능하다.   
  
```
#include <iostream>
#include <chrono>
#include <typeinfo>
#include <ctime>

template<typename Clock>
void resolution() 
{
	using namespace std;

	cout << typeid(Clock).name() << endl;

	for (int i = 0; i < 5; ++i) 
	{
		Clock::time_point t0 = Clock::now();
		Clock::time_point t1;
	
		do 
		{
			t1 = Clock::now();
		} while (t0 == t1);

		cout << chrono::duration_cast<chrono::nanoseconds>(t1 - t0).count() << " [ns]\t"
			<< chrono::duration_cast<chrono::microseconds>(t1 - t0).count() << " [micro-s]" << endl;
	}

	cout << endl;
}

int main() 
{
	using namespace std;
	chrono::system_clock::time_point time = chrono::system_clock::now();
	time_t system_time = chrono::system_clock::to_time_t(time);
	cout << ctime(&system_time) << endl;

	resolution<chrono::system_clock>();
	resolution<chrono::steady_clock>();
	resolution<chrono::high_resolution_clock>();
}
```  
  
<br>  
<br>  
    
[출처](https://codezine.jp/article/detail/9084?p=3)  