---
layout: post
title: C++ - C++03과 C++11 싱글톤 구현 차이
published: true
categories: [C++]
tags: c++ c++11 singleton
---
C++11에는 'C++11 - 블럭 범위를 가진 static 변수 초기화는 스레드 세이프 하다' 라는 기능이 생겼다.  
그래서 아래와 같이 쉽게 싱글톤 인스턴스 생성 가능하다. 
  
```
template <typename T>
inline T& Singleton<T>::getInstance() {
	static T inst;
	return inst;
}
```  
    
<br>  
  
그러나 C++03 에서는 멀티스레드에서 호출될 수 있다면 아래와 같이 해야 한다.  
```
template <typename T>
std::once_flag Singleton<T>::onceFlag;

template <typename T>
inline T& Singleton<T>::getInstance() {
	std::call_once(onceFlag, [&] {
		pInstance.reset(new T());
	});
	return *pInstance;
}
``` 
  
  