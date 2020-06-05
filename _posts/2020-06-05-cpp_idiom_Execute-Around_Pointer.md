---
layout: post
title: C++ - idiom - Execute-Around Pointer
published: true
categories: [C++]
tags: c++ idiom
---
[원문](https://qiita.com/MoriokaReimen/items/cf492903ac93c613a3e1)  
  
스마트 포인터는 객체의 라이프 타임이 다한 때에 자동적으로 확보된 리소스를 삭제한다. 
생성자와 소멸자, 연산자 오버 라이드를 잘 구현하였다.  
이것을 응용하면 클래스의 멤버 함수가 실행될 때 특정 처리를 실행하는 스마트 포인터를 구현할 수 있다.  
  
```
class ExecutePointer {
public:
    class proxy {
    public:
          proxy (vector<int> *v) : vect (v) {
              std::cout << "Before size is:" << vect->size();
          }
          vector<int> * operator -> () {
              return vect;
          }
          ~proxy () {
              std::cout << "After size is:" << vect->size ();
          }
   private:
          vector <int> * vect;
   };
   
   ExecutePointer(vector<int> *v) : vect(v) {}
   
   proxy operator -> () {
       return proxy (vect);
   }
   
private:
   vector <int> * vect;
};

int main()
{
    ExecutePointer vector(new std::vector<int>);

    vector->push_back (10);
    vector->push_back (20);

    return 0;
}
```  
    