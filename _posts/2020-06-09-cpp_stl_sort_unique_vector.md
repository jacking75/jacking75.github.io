---
layout: post
title: C++ - std::sort과 std::unique로 std::vector의 중복 요소를 삭제
published: true
categories: [C++]
tags: c++ stl sort unique vector
---
  
```
#include <iostream>   // std::cout, std::endl;
#include <algorithm>  // std::unique
#include <vector>     // std::vector

void printVec(std::vector<int> &vec) {
  std::cout << "";
  for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
  }
  std::cout << std::endl;
}

int main() {
  std::vector<int> vec = {10,40,40,20,20,30,20,20,40};
  std::unique(vec.begin(), vec.end());
  printVec(vec);    // 10 40 20 30 20 40 ? ? ?
}
```
  
std::unique는 인접한 중복 요소를 삭제하지만, vector의 길이 등에는 변경을 가하지 않기 때문에 말미에 쓰레기가 남는다.  
그래서 미리 std::sort을 사용하여 vector의 요소를 정렬한 후 std::unique를 적용한다.  
  
```
#include <iostream>   // std::cout, std::endl;
#include <algorithm>  // std::sort, std::unique
#include <vector>     // std::vector

void printVec(std::vector<int> &vec) {
  std::cout << "";
  for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
  }
  std::cout << std::endl;
}

int main() {
  std::vector<int> vec = {10,40,40,20,20,30,20,20,40};
  std::sort(vec.begin(), vec.end());
  vec.erase(std::unique(vec.begin(), vec.end()), vec.end());
  printVec(vec);    // 10 20 30 40
}

```  
  
std::unique는 쓰레기 앞의 포인터를 돌려주기 때문에 vector::erase에서 뒤의 쓰레기를 삭제한다.  
이것으로 중복 요소가 없는 vector를 얻을 수 있다.  
  
  