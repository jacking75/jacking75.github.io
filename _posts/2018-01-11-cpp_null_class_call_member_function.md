---
layout: post
title: C++ - null 객체의 멤버 함수 호출
published: true
categories: [C++]
tags: c++ c++11 _Pragma
---
클래스를 메모리 할당(new 생성)하지 않고 생성한 후 멤버 함수를 호출하면 어떻게 될까?  
호출된 멤버 함수가 자신의 멤버 변수를 호출하지 않는다면 무사히 호출된다.  
당근 멤버 변수를 호출하면 세그먼트폴이 생긴다.  
  
아래 코드는 gcc 8.0 에서 테스트 되었다. VC에서도 문제 없이 되는 것으로 안다.  
(이게 C++ 표준 규격에서는 멤버 함수를 호출 할 수 없어야 한다고 한다)  
Print()는 무사히 호출되지만 Print2()를 호출하면 세그먼트폴 발생.  
     
```
#include <iostream>

using namespace std;

class Test
{
    int mNum = 0;

public:
    void Print()
    {
        std::cout << "Test.Print" << std::endl;
    }
    
    void Print2()
    {
        std::cout << "Test.Print: " << mNum << std::endl;
    }
};

int main()
{
   cout << "Hello World" << endl; 
   
   Test *pTest = nullptr;
   pTest->Print();
   pTest->Print2();
   
    return 0;
}
```
  