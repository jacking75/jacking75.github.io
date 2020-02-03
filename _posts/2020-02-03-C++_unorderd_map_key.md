---
layout: post
title: C++ - unorderd_set, unorderd_map의 key를 임의의 클래스로 하기
published: true
categories: [C++]
tags: c++ unorderd_map
---
[출처](https://qiita.com/izmktr/items/8e0fd1b6e37de59a9bd0 )  
  
클래스의 같은 값 비교, 해시 함수가 필요  
```
#include <unordered_map>

class Point{
public:
    int x, y;
    Point(){}
    Point(int x, int y):x(x),y(y){}
    bool operator==(const Point &left) const {return this->y == left.y;}
};

namespace std{
    template<>
    class hash<Point>{
        public:
        size_t operator () ( const Point &p ) const{ return p.y;}
    };
}

int  main(){
    typedef std::unordered_map<Point, int> PointMap;
    PointMap pmap;
    pmap.insert(PointMap::value_type(Point(0,0), 0));

    return 0;
}
```
  
아래는 boost::unorderd_map   
```
#include <boost/unordered_map.hpp>
#include <unordered_map>

class Point{
public:
    int x, y;
    Point(){}
    Point(int x, int y):x(x),y(y){}
    bool operator==(const Point &left) const {return this->y == left.y;}
};

size_t hash_value(Point const& p) {
    return p.y;
}

int  main(){
    typedef boost::unordered_map<Point, int> PointMap;
    PointMap pmap;
    pmap.insert(PointMap::value_type(Point(0,0), 0));

    return 0;
}
```  
  
  
생성자에서 지정하는 방법  
```
#include <unordered_map>
#include <boost/unordered_map.hpp>

class Point{
public:
    int x, y;
    Point(){}
    Point(int x, int y):x(x),y(y){}
};

int  main(){
    typedef std::unordered_map<Point, int, size_t (*)(const Point&), bool(*)(const Point&,const Point&)> PointMap;

//  boost::unordered_map 에서도 지정 방법은 같음
//  typedef boost::unordered_map<Point, int, size_t (*)(const Point&), bool(*)(const Point&,const Point&)> PointMap;

    PointMap pmap(0, [](const Point&p)->size_t{return p.y;}, [](const Point&l,const Point&r){return l.y == r.y;});

//  decltype를 사용한 버전
//  auto h = [](const Point&p)->size_t{return p.y;};
//  auto eq = [](const Point&l,const Point&r){return l.y == r.y;};
//  typedef std::unordered_map<Point, int, decltype(h), decltype(eq)> PointMap;
//  PointMap pmap(0, h, eq);

    pmap.insert(PointMap::value_type(Point(0,0), 0));

    return 0;
}
```
  
  
boost에는 해시를 간단하게 만들 수 있는 방법이 있다.   
hash_value는 int나 string은 물론 배열, vector, tuple, map 등에도 대응한다.  
```
#include <boost/functional/hash.hpp>

size_t hash_value(const Point &p){
    using boost::hash_value;
    using boost::hash_combine;

    size_t seed = 0;
    hash_combine(seed, hash_value(p.x));
    hash_combine(seed, hash_value(p.y));
    return seed;
}
```
  