---
layout: post
title: C++ - 구조체 멤버 접근을 배열처럼 접근하고 싶은 경우
published: true
categories: [C++]
tags: c++ struct
---
일반적으로 구조체 멤버에 접근하는 예  
```
struct Point {
  int x;
  int y;
  int z;
}
 
int main(void){
 
    Point p;
    p.x = 100;
    p.y = 200;
    p.z = 300;
 
    cout << "p = (" << p.x << ", " << p.y << ", " << p.z << ")\n";
}
```  
  
  
구조체 멤버 접근을 배열 인덱스로  
```
struct Point {
    int x;
    int y;
    int z;
 
    // x의 주소를 기준으로 한 인덱스 위치 참조를 반환한다
    int& operator[] (int index) { return *(&x + index); }
};
 
int main(void){
 
    Point p;
    p.x = 100;
    p.y = 200;
    p.z = 300;
 
    cout << "p = (" << p.x << ", " << p.y << ", " << p.z << ")\n";
 
    p[0] = 10;
    p[1] = 20;
    p[2] = 30;
    cout << "p = (" << p[0] << ", " << p[1] << ", " << p[2] << ")\n";
}
```  
  
오퍼레이터를 추가하는 것이 귀찮거나 or 외부 라이브러리라서 할 수 없는 경우에는 아래와 같이 할 수도 있다.  
```
int main(void){
 
    Point p;
 
    int* p2 = &p.x;
    p2[0] = 10;
    p2[1] = 20;
    p2[2] = 30;
 
    cout << "p = (" << p.x << ", " << p.y << ", " << p.z << ")\n";
}
```
  