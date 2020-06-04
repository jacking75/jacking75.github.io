---
layout: post
title: C++ - idiom - Construction Tracker
published: true
categories: [C++]
tags: c++ idiom
---
[원문](https://qiita.com/MoriokaReimen/items/c12cd791228254a95040)  
  
생성자에서 멤버 초기화 때 예외가 발생하는 경우가 있더. try와 catch로 예외를 잡을 수는 있지만 때로는 아래와 같이 어디에서 예외가 발생했는지 알 수 없을 때가 있다.    
    
```
class A
{
public:
    A(int n)
    {

         /* 초기화에 실패해서 예외를 던진다 */
         throw std::runtime_error("A Error");
    }
};

class B
{
    B(int n)
    {
         /* 초기화에 실패해서 예외를 던진다 */
         throw std::runtime_error("B Error");
    }
};

class MyClass
{
    A a_;
    B b_;

public:
    MyClass() : a_(1), b_(2)
    {
    }
};

int main()
{
    try {
        MyClass object();
    } catch(...)
    {
     /* A의 예외? 아니면 B의 예외인가? */
    }

}
```  
  
위처럼 어느 곳에서 예외가 발생했는지 알기 위해서 Construction Tracker를 사용한다.  
```
class A
{
public:
    A(int n)
    {

         /* 초기화에 실패해서 예외를 던진다 */
         throw std::runtime_error("A Error");
    }
};

class B
{
    B(int n)
    {
         /* 초기화에 실패해서 예외를 던진다 */
         throw std::runtime_error("B Error");
    }
};

class MyClass
{
    A a_;
    B b_;
    enum TrakcerType{NONE, ONE, TWO};
public:
    MyClass(TrackerType tracker=NONE)
    try
    : a_((tracker = ONE,1)), b_((tracker = TWO, 2))
    {
        /* 정삭적으로 초기화 처리 */
    }
    catch(std::exception const& e)
    {
        if(tracker==ONE){
            /* a_ 의 초기화 실패 했을 때 처리 */
        }
    }
};

int main()
{
    try {
        MyClass object();
    } catch(...)
    {
    }

}
```  
변수 tracker를 참조해서 어떤 멤버 변수가 초기화에 실패했는지 알 수 있다.  
