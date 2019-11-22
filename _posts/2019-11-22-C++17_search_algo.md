---
layout: post
title: C++17 - Boyer-Moore, Boyer-Moore-Horspool 검색 알고리즘 추가
published: true
categories: [C++]
tags: c++ c++17 검색 search Boyer-Moore Boyer-Moore-Horspool
---
헤더 파일은  
```
#include <functional>
```  
  
  
## Boyer-Moore
- 최악의 케이스가 o(n). n은 검색 대상 범위의 길이  
- 전처리 시에 테이블을 2개 만들므로 전처리 시간이 걸리고, 메모리를 더 먹는다.
- 검색 패턴이나 검색 대상 범위가 긴 경우, 혹은 같은 검색 패턴에서 몇 번이라도 검색하는 경우에는 유리.
- 임의 접근 반복자가 필요.
  
```
boyer_moore_searcher( RandomIt1 pat_first,
                      RandomIt1 pat_last,
                      Hash hf = Hash(),
                      BinaryPredicate pred = BinaryPredicate());

template< class RandomIt2 >
std::pair<RandomIt2,RandomIt2> operator()( RandomIt2 first, RandomIt2 last ) const;
```
  
[출처](https://en.cppreference.com/w/cpp/utility/functional/boyer_moore_searcher )  
```
#include <iostream>
#include <string>
#include <algorithm>
#include <functional>
 
int main()
{
    std::string in = "Lorem ipsum dolor sit amet, consectetur adipiscing elit,"
                     " sed do eiusmod tempor incididunt ut labore et dolore magna aliqua";
    std::string needle = "pisci";
    auto it = std::search(in.begin(), in.end(),
                   std::boyer_moore_searcher(
                       needle.begin(), needle.end()));
    if(it != in.end())
        std::cout << "The string " << needle << " found at offset "
                  << it - in.begin() << '\n';
    else
        std::cout << "The string " << needle << " not found\n";
}
```
  
Output:  
<pre>
The string pisci found at offset 43
</pre>
  
  
  
## Boyer-Moore-Horspool  
- 최악의 케이스는 대부분 O(n)  
- 전처리 시에 만드는 테이블은 1개이므로 Boyer-Moore법 보다는 전처리 시간도 메모리도 덜 먹는다  
- 임의 접근 반복자가 필요.
  
```
template< class RandomIt1,
          class Hash = std::hash<typename std::iterator_traits<RandomIt1>::value_type>,
          class BinaryPredicate = std::equal_to<> >
class boyer_moore_horspool_searcher;

template< class RandomIt2 >
std::pair<RandomIt2,RandomIt2> operator()( RandomIt2 first, RandomIt2 last )
```
  
[출처](https://en.cppreference.com/w/cpp/utility/functional/boyer_moore_horspool_searcher )   
```
#include <iostream>
#include <string>
#include <algorithm>
#include <functional>
 
int main()
{
    std::string in = "Lorem ipsum dolor sit amet, consectetur adipiscing elit,"
                     " sed do eiusmod tempor incididunt ut labore et dolore magna aliqua";
    std::string needle = "pisci";
    auto it = std::search(in.begin(), in.end(),
                   std::boyer_moore_horspool_searcher(
                       needle.begin(), needle.end()));
    if(it != in.end())
        std::cout << "The string " << needle << " found at offset "
                  << it - in.begin() << '\n';
    else
        std::cout << "The string " << needle << " not found\n";
}
```
  
Output:  
<pre>
The string pisci found at offset 43
</pre>  
  
  
