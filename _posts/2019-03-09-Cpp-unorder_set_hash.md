---
layout: post
title: unorder_set 에서 구조체(혹은 클래스)를 Key로 하고 싶을 때
published: true
categories: [C++]
tags: c++ unorder_set hash
---
[출처](https://qiita.com/zhupeijun/items/1d90ad5ef7651048fcc5 )  
  
- == 오퍼레이터를 오버라이드 한다.  
- hash 함수를 구현한다.  
  
```
// key로 사용할 구조체
struct Point {
    int x, y;

    Point(int x, int y) : x(x), y(y) { }
	
    // override == operator
    bool operator == (const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// P:oint 구조체를 사용하는 해시 함수
namespace std {
    template<>
    struct hash<Point> {
        size_t operator () (const Point& point) {
            // just a simple way to hash a point range between 0..100
            return point.x * 100 + point.y;
        }
    };
}
```
  
  
```
int main() {
    unordered_set<Point> point_set;
    point_set.insert(Point(0,0));
    point_set.insert(Point(0,1));
    point_set.insert(Point(0,1));

    for (auto& point : point_set) {
        cout << point.x << " " << point.y << endl;
    }
    return 0;
}
```
  
