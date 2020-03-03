---
layout: post
title: C++ - boost::irange
published: true
categories: [C++]
tags: c++ boost
---
[출처](https://qiita.com/Lily0727K/items/ab089a4974c614e7f861 )   
  
정수의 범위를 사용하려면 boost::irange 함수를 사용한다.  
  
범위로서 `[0, n)` 을 지정하고, 제 3인수로 단계 폭을 2로 지정하고 있다. (제 3 인수를 생략하면 단계 폭은 1이 된다)  
이것에 대해 range-based for 문을 작성할 수 있으며, Python의 for 문과 비슷하다.  
  
  
```
#include <bits/stdc++.h>
#include <boost/range/irange.hpp>

using namespace std;
using boost::irange;

int main() {
  string s;
  cin >> s;
  int n = s.size();
  for (int i : irange(0, n, 2)) {
    cout << s[i];
  }
  cout << endl;
}
```
  
