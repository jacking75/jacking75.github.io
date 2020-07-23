---
layout: post
title: C++ - 함수 템플릿의 포인터를 함수로 받기
published: true
categories: [C++]
tags: c++ template
---
[출처](https://osyo-manga.hatenadiary.org/entry/20120307/1331097297  )  
  
C++11 이상 가능.  
```
// 템플릿의 default 값에 함수 포인터 타입을 정의
template<typename T, typename Func = T(*)(T, T)>
constexpr T
calc(T a, T b, Func func){
    return func(a, b);
}

template<typename T>
constexpr T
plus(T a, T b){
    return a + b;
}

struct minus{
    template<typename T>
    constexpr T
    operator ()(T a, T b) const{
        return a - b;
    }
};


int main()
{
    static_assert(plus(1, 2) == 3, "");
    static_assert(calc(1, 2, &plus<int>) == 3, "");
    static_assert(calc(1, 2, &plus) == 3, "");
    static_assert(calc(1, 2, minus{}) == -1, "");   // ok
    return 0;
}
```
  
