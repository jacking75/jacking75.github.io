---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) cheked delete
published: true
categories: [C++]
tags: c++ idiom chekeddelete
---
규격에 의해 delete식에 의해 불완전 타입을 delete에 넘기는 것이 허가되고 있다.   
그러나 trival 생성자가 아닌지, 독자적인 delete 연산자를 정의하고 있는 경우 미 정의 동작이 된다.  
미정의 동작은 좋지 않으므로 delete 대상 형식이 불완전 타입인 경우 컴파일러에 에러를 내자는 idiom 이다.  
  
```
template <class T>
inline void cheked_delete(T*& p)
{
    typedef char type[sizeof(T)?1:-1];
    (void)sizeof(type);    
    delete p;
    p=nullptr;
}
```  
  
불완전 타입을 받았을 때 배열의 요소 수를 음수로 하고 typedef 하기 때문에 에러가 되게 한다는 방법이다.  
```
#include <boost/utility.hpp>

class Hoge;
Hoge* CreateHoge();

int main()
{
    Hoge* ptr = CreateHoge();

    ...

    boost::checked_delete( ptr ); // 여기에서 컴파일 에러

    return 0;
}
```  
  