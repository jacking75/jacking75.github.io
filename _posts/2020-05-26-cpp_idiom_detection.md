---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) Detection
published: true
categories: [C++]
tags: c++ idiom Detection
---
C++17에서 type_traits 헤더에 추가된 std::void_t를 이용한 idiom 이다.  
std::void_t의 구현은 아래처럼 되어 있다.  
  
```
template<class...>
using void_t=void;
```
  
뭘 넣어도 void를 반환하므로 넢고 싶은 것을 마음대로 넣을 수 있다.  
아래의 예는 클래스가 반복자를 가지고 있는가를 조사하는 것이다.  
  
```
template<class,class=std::void_t<>>
constexpr bool has_iterator_v=std::false_type::value;

template<class Tp>
constexpr bool has_iterator_v<Tp,std::void_t<typename Tp::iterator::iterator_category>> =std::true_type::value;

int main()
{
    std::cout << std::boolalpha<<has_iterator_v<std::vector<int>> << std::endl; // true
}

```