---
layout: post
title: C++11 - 블럭 범위를 가진 static 변수 초기화는 스레드 세이프 하다
published: true
categories: [C++]
tags: c++ c++11 static
---
### 개요

static 변수의 초기화가 완료할 때까지 다른 스레드는 초기화 처리 앞에서 대기한다.

```
class singleton {
public:
  static singleton& get_instance()
  {
    static singleton instance; // 이 초기화는 스레드 세이프
    return instance;
  }
};
```

get_instance 함수를 복수의 스레드에서 호출할 때 singleton 객체 초기화는 하나의 스레드에서만 호출된다. 

C++11 이전에는 로컬의 static 변수를 초기화 하는 것이 스레드 세이프하지 않았기 때문에 Double Checked Locking 라는 기법을 사용하였다.  
<br> 
<br>  

 
### Double Checked Locking
- static 지역 변수 대신 포인터를 static 멤버 변수로 가진다
- 포인터가 null 인지 아닌지를 체크하고, null 이라면 lock을 획득한다.
- null 체크 및lock 취득 사이에 유효한 포인터가 될 가능성이 있기 때문에 lock 취득 후 한번 더 null 체크를 하고 null 이면 초기화를 한다






