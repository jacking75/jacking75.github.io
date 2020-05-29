---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) No Copyable mix-in
published: true
categories: [C++]
tags: c++ idiom Copyable
---
계승된 클래스는 복사 금지라고 명쾌하게 표현할 수 있는 idiom 이다.  
  
```
class noCopyable{
    noCopyable(const noCopyable&)=delete;
    noCopyable& operator=(const noCopyable&)=delete;
protected:
    noCopyable()=default;
    ~noCopyable()=default;
};
struct X:private noCopyable{};
```  
  
  
CRTP를 이용한 구현  
```
template<class>
class noCopyable{
    noCopyable(const noCopyable&)=delete;
    noCopyable& operator=(const noCopyable&)=delete;
protected:
    noCopyable()=default;
    ~noCopyable()=default;
};
struct X:private noCopyable<X>{};
struct Y:private noCopyable<Y>{};
struct Z:X,Y{};

int main()
{
    std::cout<<sizeof(Z)<<std::endl; // 1
}
```  
  
  