---
layout: post
title: C++ - 각종 STL 컨테이너에서 루프 중 요소 삭제하기
published: true
categories: [C++]
tags: c++ stl
---
[원문](https://qiita.com/satoruhiga/items/fa6eae09c9d89bd48b5d   )  
  
## vector
  
```
vector<int> v;
v.push_back(10);
v.push_back(20);
v.push_back(30);

vector<int>::iterator it = v.begin();
while (it != v.end()) {
    if (*it == 20) {
        it = v.erase(it);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << *it << endl;
    ++it;
}
```
  
  
## deque
  
```
deque<int> v;
v.push_back(10);
v.push_back(20);
v.push_back(30);

deque<int>::iterator it = v.begin();
while (it != v.end()) {
    if (*it == 20) {
        it = v.erase(it);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << *it << endl;
    ++it;
}
```
  
  
## list
  
```
list<int> v;
v.push_back(10);
v.push_back(20);
v.push_back(30);

list<int>::iterator it = v.begin();
while (it != v.end()) {
    if (*it == 20) {
        it = v.erase(it);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << *it << endl;
    ++it;
}
```
  
  
## map
  
```
map<int, int> v;
v[10] = 10;
v[20] = 20;
v[30] = 30;

map<int, int>::iterator it = v.begin();
while (it != v.end()) {
    if (it->second == 20) {
        v.erase(it++);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << it->first << " " << it->second << endl;
    ++it;
}
```
  
  
## multimap
  
```
multimap<int, int> v;
v.insert(make_pair(10, 10));
v.insert(make_pair(20, 20));
v.insert(make_pair(30, 30));

multimap<int, int>::iterator it = v.begin();
while (it != v.end()) {
    if (it->second == 20) {
        v.erase(it++);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << it->first << " " << it->second << endl;
    ++it;
}
```
  
  
## set
  
```
set<int> v;
v.insert(10);
v.insert(20);
v.insert(30);

set<int>::iterator it = v.begin();
while (it != v.end()) {
    if (*it == 20) {
        v.erase(it++);
    } else ++it;
}

it = v.begin();
while (it != v.end()) {
    cout << *it << endl;
    ++it;
}
```
  
  
vector와 deque 같은 시퀸스 컨테이너들은 erase의 반환 값이 반복자이므로 이것이 다음의 반복자가 된다.   
map, set 같은 연관 컨테이너들은 erase 한 반복자는 무효한 주소를 가리키므로 it++을 넘겨서 반복자를 복사하면서 요소를 삭제하고, 반복자를 진행시킨다. -> 복사된 반복자를 사용하여 연관 컨테이너의 요소를 삭제한다.  
  
 