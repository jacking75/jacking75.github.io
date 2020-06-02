---
layout: post
title: C++ - idiom - Construct on First Use
published: true
categories: [C++]
tags: c++ idiom
---
[원문](https://qiita.com/MoriokaReimen/items/c12cd791228254a95040)
정적 오브젝트는 주의하지 않으면 초기화 되기 전에 사용하는 실수를 할 수 있다. 아래는 그 예이다.  
    
```
#include <iostream>
class StaticObject
{
public:
    StaticObject()
    {
        std::cout << "StaticObject::StaticObject()" << std::endl;
    }

    void callMethod()
    {
        std::cout << "StaticObject::callMethod()" << std::endl;
    }
};

class MyClass
{
    static StaticObject object_;
public:
    MyClass()
    {
        std::cout << "MyClass::MyClass()" << std::endl;
        object_.callMethod();
    }
};

MyClass my_class; // ! 초기화 되지 않은 object 에 접근
StaticObject MyClass::object_;

int main()
{
    std::cout << "main" << std::endl;
}
'''  
  
이런 문제를 해결하기 위해 정적 오브젝트로의 접근을 함수로 통해서 하도록 한다.  
```
#include <iostream>
class StaticObject
{
public:
    StaticObject()
    {
        std::cout << "StaticObject::StaticObject()" << std::endl;
    }

    void callMethod()
    {
        std::cout << "StaticObject::callMethod()" << std::endl;
    }
};

class MyClass
{
    StaticObject& Object()
    {
         static StaticObject object_;
         return object_;
    }
public:
    MyClass()
    {
        std::cout << "MyClass::MyClass()" << std::endl;
        Object().callMethod();
    }
};

MyClass my_class; 

int main()
{
    std::cout << "main" << std::endl;
}
```  
  