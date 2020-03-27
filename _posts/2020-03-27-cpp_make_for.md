---
layout: post
title: C++ - 유저 정의 클래스에서 간단하게 확장 for 문을 사용할 수 있도록 하는 방법
published: true
categories: [C++]
tags: c++ class for
---
[출처](https://qiita.com/_meki/items/ceecfe529fc5697de8ab )  
  
## STL 컨테이너를 public 상속한 경우
유의해야 할 것은 STL 컨테이너의 소멸자는 virtual 이 아니다  
예를 들면 부모 클래스의 포인터로 업 캐스트하고 delete 하면 자식 클래스의 소멸자가 호출되지 않는다.  
  
코드 예  
```
#include <iostream>
#include <vector>
using namespace std;

using VectReal = std::vector<double>;

class MyVect : public VectReal
{
public:
    MyVect(std::initializer_list<double> list)
        : VectReal(list)
    { }
};

int main(void)
{
    MyVect v = { 1, 2, 3, 4, 5 };
    for (double d : v) { cout << d << " "; }
}
```
  
콘솔 출력  
1 2 3 4 5  
  
  
  
## STL 컨테이너를 private 상속한 경우
부모 클래스의 필요한 멤버를 using 선언을 추가한다.  
  
코드 예  
```
#include <iostream>
#include <vector>
using namespace std;

using VectReal = std::vector<double>;

class MyVect : private VectReal
{
public:
    /* 부모 클래스의 반복자를 사용하기 위해 iterator, const_iterator, begin, end 을 using 한다 */
    using VectReal::iterator;
    using VectReal::const_iterator;
    using VectReal::begin;
    using VectReal::end;
    // ...이 외 사용하고 싶은 메소드, 멤버, typedef 를 using 한다

    MyVect(std::initializer_list<double> list)
        : VectReal(list)
    { }
};

int main(void)
{
    MyVect v = { 1, 2, 3, 4, 5 };
    for (double d : v) { cout << d << " "; }
}
```
  
콘솔 출력  
1 2 3 4 5  
  
  
  
## STL 컨테이너를 가진 경우
  
코드 예  
```
#include <iostream>
#include <vector>
using namespace std;

using VectReal = std::vector<double>;

class MyVect
{
public:

    MyVect(std::initializer_list<double> list)
        : m_vect(list)
    { }

    // iterator, const_iterator의 typedef를 해 두면 MyVect::iterator 라고 쓸 수 있어서 편리하다(필수는 아니다)
    using iterator = VectReal::iterator;
    using const_iterator = VectReal::const_iterator;

    // m_vect 로 처리를 위임
    iterator begin() { return m_vect.begin(); }
    iterator end() { return m_vect.end(); }
    const_iterator begin() const { return m_vect.begin(); }
    const_iterator end() const { return m_vect.end(); }


private:
   // 멤버로 가진다
   VectReal m_vect;
};

int main(void)
{
    MyVect v = { 1, 2, 3, 4, 5 };
    for (double d : v) { cout << d << " "; }
}
```
  
콘솔 출력  
1 2 3 4 5
  
  
  
## 포인터와 배열을 멤버로 가진 경우
① 포인터를 반복자로서 typedef 하고, ② begin, end를 아래처럼 구현한다  
rbegin, rend 도 같은 요령으로 구현할 수 있다.  
  
코드 예  
```
#include <iostream>
#include <vector>
using namespace std;

class MyVect
{
public:

    MyVect(std::initializer_list<double> list)
    {
        m_size = list.size();
        m_vect = new double[m_size];
        std::copy(list.begin(), list.end(), m_vect);
    }

    using iterator = double*;
    using const_iterator = const double*;

    iterator begin() { return m_vect; }
    iterator end() { return m_vect + m_size; }
    const_iterator begin() const { return m_vect; }
    const_iterator end() const { return m_vect + m_size; }


private:
   double* m_vect;
   size_t m_size;
};

int main(void)
{
    MyVect v = { 1, 2, 3, 4, 5 };
    for (double d : v) { cout << d << " "; }
}
```
  