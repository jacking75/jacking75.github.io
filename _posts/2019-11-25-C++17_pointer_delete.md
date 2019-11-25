---
layout: post
title: C++17 - null pointer를 delete 해도 문제 없음 
published: true
categories: [C++]
tags: c++ c++17 pointer delete
---
C++14까지는 메모리 할당을 받은 객체인지 확인 후 delete를 하였다.
```
if(pUser) {
    delete pUser;
}
```
  
C++17 에서는 null pointer를 delete 해도 문제 없다(아래 설명에서는 C++14도 괜찮은 듯...).  
```
delete pUser;
```
  
http://en.cppreference.com/w/cpp/language/delete    
> If expression evaluates to a null pointer value, no destructors are called, and the deallocation function may or may not be called (it's implementation-defined), but the default deallocation functions are guaranteed to do nothing when handed a null pointer. (until C++14)
> 
> If expression evaluates to a null pointer value, no destructors are called, and the deallocation function is not called. (since C++14)

expression이 널 포인터 값으로 평가된 경우 소멸자는 호출 되지 않는다. 해방 함수는 호출 될 수도 있고, 호출 되지 않을 수도 있지만(컴파일러에서 구현하기 나름), default 해방 함수는 널 포인터가 전달될 때 아무것도 하지 않는 것이 보증 되고 있다.(C++14 미만)  
  
expression이 널 포인터 값으로 평가된 경우 소멸자는 호출되지 않고, 해방 함수도 호출되지 않는다.(C++14 이상)    
      