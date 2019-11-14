---
layout: post
title: C++17 - 람다식에서 *this의 복사 캡쳐
published: true
categories: [C++]
tags: c++ c++17 lambda
---
C++11에서 람다식의 캡쳐 리스트에 this를 지정하면 이 람다식이 속하는 클래스 오브젝트가 참조로 캡쳐된다.  
  
비동기 처리나 병렬 처리에서 오브젝트 수명 관리를 간단하게 하기 위해 this를 참조가 아닌 복사로 캡쳐해야 하는 경우가 있다.  
이 때 C++17에서 새로 생긴 *this 를 사용하여 클래스 오브젝트를 복사로 캡쳐할 수 있다.  
  
[온라인 코드 실행](https://wandbox.org/permlink/XecsgBmUSLBgVu9L)    
```
#include <iostream>

struct Test {
    int N1 = 7;
    
    auto AddNum1()
    {
        // 복사 캡쳐이므로 N1의 값은 스코프를 벗어나면 원래대로 돌아간다
        [*this]() mutable { N1 += 10; }();
    }
    
    auto AddNum2()
    {
        // 참조 캡쳐이므로 N1의 값은 변경된다
        [this]() { N1 += 10; }();
    }
};

int main()
{
    Test test1;
    test1.AddNum1();   
    std::cout << test1.N1 << std::endl;
    
    Test test2;
    test2.AddNum2();    
    std::cout << test2.N1 << std::endl;
}
```  
  
< 결과 >
7
17  
  
<br>  
  
[=], [&]에 의한 default 캡쳐에서는 this는 참조로 캡쳐된다.   
그리고 [this, *this]는 당연히 안 된다.  
        