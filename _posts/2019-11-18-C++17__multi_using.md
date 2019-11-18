---
layout: post
title: C++17 - 복수의 using 선언 시 컴마 구분 사용 
published: true
categories: [C++]
tags: c++ c++17 using
---
간결한 코딩을 위해 네임 스페이스를 생략하고 싶다면 using 선언을 사용한다  
```
std::cout << "hello";
```
```
using std::cout;

cout << "hello";
```
   
그러나 아래와 같은 경우는   
```
using std::cout;
using std::endl;

cout << "hello" << endl;
```  
로 두 번이나 using  선언을 해야 한다.  
  
C++17에서는 위의 경우를 좀 더 간편하게 선언할 수 있다.  
```
using std::cout, std::endl; 

cout << "hello" << endl;
}
```
    
      