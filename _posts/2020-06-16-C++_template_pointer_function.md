---
layout: post
title: C++ - 함수 템플릿 포인터를 함수에서 파라미터로 받기
published: true
categories: [C++]
tags: c++ template pointer
---
[출처](https://osyo-manga.hatenadiary.org/entry/20120307/1331097297 )  
  
함수 템플릿의 포인터를 함수에 넘기는 경우  
```
template<typename T, typename Func>
T
calc(T a, T b, Func func){
    return func(a, b);
}

template<typename T>
T
plus(T a, T b){
    return a + b;
}

assert(calc(1, 2, &plus<int>) == 3, "");   // ok
assert(calc(1, 2, &plus) == 3, "");        // error
```
  
위처럼 호출 측에서 명시적으로 타입을 정의하고 넘겨야 하지만, 받는 측에서 함수 포인터를 정의해 두면 문제 없이 넘길 수 있다.  
  
```
// 템플릿의 기본 값에 함수 포인터 타입을 정의
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

int
main(){
    static_assert(plus(1, 2) == 3, "");
    static_assert(calc(1, 2, &plus<int>) == 3, "");
    static_assert(calc(1, 2, &plus) == 3, "");
    static_assert(calc(1, 2, minus{}) == -1, "");   // ok
    return 0;
}
```
  
  