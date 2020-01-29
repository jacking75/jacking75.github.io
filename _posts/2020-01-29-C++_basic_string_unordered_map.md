---
layout: post
title: C++ - 커스텀 Allocator를 사용하는 basic_string을 unordered_map의 key로 사용하는 방법
published: true
categories: [C++]
tags: c++ basic_string unordered_map
---
아래의 코드는 컴파일 에러가 된다.  
```
using MyString = std::basic_string<
                     char, 
                     std::char_traits<char>,
                     MyAllocator<char>>;


std::unordered_map<MyString, int> umap; // 이 부분이 에러
```  
이유는 unordered_map이 필요로 하는 std::hash<T>는 std::hash<std::string>로 정의 되어 있는데 allocator를 특별화한 문자열 타입의 정의가 없기 때문이다.  

C++17에서는 이 문제를 간단하게 해결할 수 있다.  
std::hash<std::string_view>를 사용하면 된다.  
```
template <>
struct std::hash<MyString>
{
    std::size_t operator()(const MyString& k) const
    {
        const std::string_view sv{k.c_str(), k.length()};
        return std::hash<std::string_view>()(sv);
    }
};
```
  
C++14 의 경우는 boost 라이브러리를 사용하면 쉽게 해결 가능하다.  
```
#include <boost/functional/hash.hpp>

template <>
struct std::hash<MyString>
{
    std::size_t operator()(const MyString& k) const
    {
        return boost::hash_range(k.begin(), k.end());
    }
};
```
  
  
만약 C++17, C++14 모두 사용할 수 없다면 해시 함수를 만들어야 한다.  
이 때 `const char*` 로 하여 표준의 `std::hash<T*>` 를 사용하면 안 된다.   
`std::hash<T*>` 는 포인터의 어드레스 값으로만 해시를 계산하기 때문이다.  
다른 어드레스에 있는 동일한 문자열이 다른 해시 값이 되어 버린다.  
  
Fowler–Noll–Vo hash function 을 추천한다.  
Wikipesia 에도 The FNV hash was designed for fast hash table and checksum use, not cryptography. 라고 써여 있을 정도로 좋다.  
아래처럼 사용한다.  
```
#if SIZE_MAX == UINT32_MAX
constexpr std::size_t FNV_PRIME = 16777619u;
constexpr std::size_t FNV_OFFSET_BASIS = 2166136261u;
#elif SIZE_MAX == UINT64_MAX
constexpr std::size_t FNV_PRIME = 1099511628211u;
constexpr std::size_t FNV_OFFSET_BASIS = 14695981039346656037u;
#else
#error "size_t must be greater than 32 bits."
#endif

template <>
struct std::hash<MyString>
{
    std::size_t operator()(const MyString& k) const
    {
        std::size_t hash = FNV_OFFSET_BASIS;
        for(auto it = k.begin(); it != k.end(); it++) {
            hash ^= *it;
            hash *= FNV_PRIME;
        }
        return hash;
    }
};
``` 
  
  
<br>
    
[출처](https://qiita.com/nakat-t/items/e5eed0e67b5cb74f8f59 )