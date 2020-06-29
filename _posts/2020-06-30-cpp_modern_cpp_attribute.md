---
layout: post
title: C++ - 표준 attribute 정리
published: true
categories: [C++]
tags: c++ attribute noreturn carries_dependency deprecated fallthrough maybe_unused nodiscard no_unique_address likely unlikely
---
[원문](https://qiita.com/tyanmahou/items/4ebd03f203a5a8594cf7   )  
  
## attribute(속성) 라는 것은
attribute(속성)는 컴파일러에 추가 정보를 전달하는 구문으로 `[[attributes]]` 이라고 쓴다.  
최적화나 경고 추가나 제어 등을 할 수 있다.  
  
  
## C++11
  
### noreturn 속성
함수  
  
함수가 결코 반환 하지 않는 것을 표시하는 속성  
예외 송출이나 `std::exit`, `std::abort` 의 랩퍼 함수에 붙인다.  
이것에 의해 함수가 반환하지 않는 패스가 있는 경우의 경고를 억제한다.  
```
[[noreturn]] void exit()
{
    std::exit(0);
}
int hoge(int x)
{
    if (x > 0) {
        return x;
    } // if에 들어가지 않는 경우 return이 없지만
    exit(); // 이 함수는 반환하지 않는다.
}
```
  
  
### carries_dependency 속성
함수, 함수 파라메터

병렬 프로그래밍의 atomic 조작에 있어서 값에 의존하는 순서를 함수 전반에 전파하는 것을 나타내는 속성이다.   
```
struct Hoge 
{
    int hoge = 0;
};
std::atomic<Hoge*> x(nullptr);

[[carries_dependency]] Hoge* is_load()
{
    return x.load(std::memory_order_consume);
}

int main()
{
    std::thread th([]() {
        x.store(new Hoge{ 100 }, std::memory_order_release);
    });

    Hoge* pHoge = is_load();
    if (pHoge)
    {
        std::cout << pHoge->hoge;
    }
    th.join();
    return 0;
}
```  
  
  
  
## C++14
  
### deprecated 속성
클래스, 타입 별칭, 열거자, 함수, 템플릿 특수화, 변수, 비 정적 멤버 변수, 열거 형(C++17), 네임 스페이스(C++17)  
  
대상의 기능이 비추천인 것을 나타내는 속성.  
API 개발 시 등에서 사용 되지 않는 기능 사용하면 사용자에게 경고로 나타낼 수 있다.  
인수로 표시하는 경고문을 지정할 수 있다.
`deprecated("reason")`  
  
```  
[[deprecated("please use new_func() function")]]
void old_func(){}
void new_func(){}

int main()
{
    old_func(); // warning
    return 0;
}
```
  
  
  
## C++17
  
### fallthrough 속성
switch-case  
  
switch문에서 통과하는 것을 명시하는 속성.  
의도적인 통과 장소에 지정하는 것으로 경고를 억제한다.  
```
int hoge(int i)
{
    int ret = 0;
    switch (i)
    {
    case 1: 
        ret += 1;
        [[fallthrough]]; // 의도적인 통과
    case 2:
        ret += 2; 
        break;
    }
    return ret;
}
```
  
  
### maybe_unused 속성
클래스, 타입 별칭, 열거타입, 함수, 함수 매개 변수, 변수, 비 정적 멤버 변수, 열거자  
  
사용하지 않는 것을 명시하는 속성.  
의도적으로 사용 되지 않는 것에 지정하여 경고를 억제한다.  
```
void hoge([[maybe_unused]]int _unused)
{
    // _unused는 사용되지 않는다
}
```
  
  
### nodiscard 속성
함수  
  
함수의 반환 값을 파기해서는 안 되는 것을 나타내는 특성.  
파기된 경우 경고문이 나온다.  
```
[[nodiscard]]int * create()
{
    return new int(0);
}

int main()
{
    create(); // warning
    return 0;
}
```
  
  
  
## C++20
  
### no_unique_address 속성
비 정적 멤버 함수

데이터 멤버가 이 클래스의 다른 모든 비 정적 데이터 멤버와 서로 다른 주소를 가질 필요가 없다는 것을 나타내는 특성  
``` 
struct Empty{};

struct Test
{
    int x;
    Empty _noused;
};

struct Test2
{
    int x;
    [[no_unique_address]] Empty _noused;
};

int main()
{
    static_assert(sizeof(Empty) >= 1);

    static_assert(sizeof(Test) >= sizeof(int) + 1);

    static_assert(sizeof(Test2) == sizeof(int));

    return 0;
}
```
  
  
### likely, unlikely 속성
if switch-case

조건 분기에서 빈도를 나타내는 속성.  
분기가 선택 되기 쉬운 경우는 likey, 선택 되기 어려운 경우는 unlikey를 지정하는 것으로 최적화 힌트를 준다.  
```
    if ([[likely]] normal())
    {
         
    }
    if ([[unlikely]] rare())
    {
        
    }
```
  
  
  
