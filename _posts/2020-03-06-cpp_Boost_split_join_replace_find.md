---
layout: post
title: C++ - split, join, replace_all, brent_find_minima
published: true
categories: [C++]
tags: c++ boost 
---
[출처](https://qiita.com/Lily0727K/items/ab089a4974c614e7f861 )   
  
## split
지정한 구분 문자로 문자열을 분할 할 때는 boost::algorithm::split 함수를 사용한다.  
얻어진 결과는 지정한 컨테이너에서 받는다.  
구분 문자인지 판정하는 것은 boost::is_any_of를 사용한다.  
아래 예는 문자열을 + 로 구분하고 있다.  
`split(plus_split_s, s, [](char c){return c == '+';});`    
  
```
#include <bits/stdc++.h>
#include <boost/algorithm/string/classification.hpp> // is_any_of
#include <boost/algorithm/string/split.hpp>

using namespace std;
using boost::algorithm::split;

int main() {
  string s;
  cin >> s;
  vector<string> plus_split_s;
  split(plus_split_s, s, boost::is_any_of("+"));
  int cnt = 0;
  for (const string& elem : plus_split_s) {
    if (elem.find('0') == string::npos) ++cnt;
  }
  cout << cnt << endl;
}
```
  
  
  
## boost::algorithm::join
컨테이너의 문자열을 연결해서 하나의 문자열로 만들 때는 boost::algorithm::join 함수를 사용한다.  
연결하는 문자열에 구분자를 지정할 수 있고, 아래는 _ 를 구분자로 하였다.   
`string s = join(arr, "_");`  
  
```
#include <bits/stdc++.h>
#include <boost/algorithm/string/join.hpp>

using namespace std; 
using boost::algorithm::join;

int main() {
  int n, l;
  cin >> n >> l;
  vector<string> arr(n);
  for (int i = 0; i < n; ++i) {
    cin >> arr[i];
  }
  sort(arr.begin(), arr.end());
  string s = join(arr, "");
  cout << s << endl;
}
```
  
  
  
## boost::algorithm::replace_all
문자열 중의 요소를 모두 다른 요소로 치환할 때는 boost::algorithm::replace_all 함수를 사용한다.  
이것을 원본을 변경한다.  
원본을 변경하지 않을 때는 boost::algorithm::replace_all_copy 함수를 사용한다.  
`string result = boost::algorithm::replace_all_copy(s, "AtCoder", "A");`  
    
```
#include <bits/stdc++.h>
#include <boost/algorithm/string/replace.hpp>

using namespace std;
using boost::algorithm::replace_all;

int main() {
  string s;
  getline(cin, s);
  replace_all(s, "Left", "<");
  replace_all(s, "Right", ">");
  replace_all(s, "AtCoder", "A");
  cout << s << endl;
}
```
  
  
  
## boost::math:: tools::brent_find_minima
함수의 최소 값을 찾을 때는 boost::math::tools::brent_find_minima 함수를 사용한다.  
  
```
#include <bits/stdc++.h>
#include <boost/math/tools/minima.hpp>

using namespace std;
using boost::math::tools::brent_find_minima;

int main() {
  double p;
  cin >> p;
  auto r = brent_find_minima([p](double x) {return x + p / pow(2.0 , x / 1.5);}, 0.0, 100.0, 30);
  cout << fixed << setprecision(15);
  cout << r.second << endl;
}
```
  
