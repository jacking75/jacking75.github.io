---
layout: post
title: C++ - 비 public 계승을 사용하는 곳
published: true
categories: [C++]
tags: c++
---
출처: 웹 어딘가에서...  
  
계승에는 3가지 단계가 있다.
- public
- protected
- private
  
객체 지향적 사고로는 public 계승은 자주 is-a 관계를 나타낸다 라고 한다.  
한편 private 계승, protected 계승은 is-a 관계가 아니다.  
is-a 관계와 대비해서 has-a 관계 또는 is-implemented-in-terms-of 관계라고 불리기도 한다.  
public 계승 이외는 B is a A 의 관계가 없기 때문에 A의 포인터에 B를 대입할 수 없다.    
```
class A{};
class B:A{};
 
int main()
{
  A*=new B; // 에러! cannot cast ‘B‘ to its private base class 'A'
  return 0;
}
```  
  
  
기저 클래스의 포인터에 대입을 허용하는 것은 public 계승 뿐이다.  
  
public 계승 이외의 계승이 기저 클래스의 멤버에는 내부에서만 접근할 수 있다. 그래서 어떤 클래스를 private 계승하고 있지만 멤버로서도 갖고 있지만 B라는 클래스의 구현의 세부를 위해서 내부에서 A의 기능을 사용한다는 상태는 사실은 바뀌지 않는다.  
  
즉 어느 클래스가 가진 기능을 사용하고 싶은 경우에는  
1. 멤버로서 갖는다  
2. 계승  
  
라는 2가지 선택지가 있다.  
  
기본적으로 계승은 코드의 복잡성을 더해서 피할 수 있으면 피하는 것이 바람직하다고 여겨지고 있다.  
최대한 콤퍼지션(내포)를 사용하여 구현하는 것이 좋다.  
  
그래도 계승을 사용해서 구현하는 상황은 무엇일까?  
하나는 EBO(Empty Base Optimization)가 되는 경우가 있다는 점을 들 수 있다. 이는 메모리 효율 관점에서 보면 큰 어드밴티지가 된다.  
  
두 번째는 당신이 라이브러리의 설계자로 자작하는 클래스 B는 A라는 클래스의 가상 함수를 오버라이드 해서 뭔가 하고 싶지만 이용자에게는 자신이 만든 클래스를 A의 포인터로서 관리되지 않기 바랄 때 라는 경우이다.  
```
#include<iostream>
 
// 원래 있던 클래스 A
class A{
public:
void do3(){ 
    do();
    do();
    do();
}
//이 함수를 오버라이드 해서 원하는 행동을 하면 좋다
virtual void do(){
    std;cout<<"A-do"<<std::endl;
}
};
 
//클래스 A의 구현을 일부 커스텀마이즈 해서 구현하고 싶은 B
class B : A{
public:
void do3(){
    A::do3();
  }

private:
// do의 새로운 거동은 이것이다!
void do()override{
    std;cout<<"B-do"<<std::endl;
  }
};
 
 
int main(){
B b;
b.do3();
return 0;
}
```  
이것으로 화면에는 "B-do라는 것이 3번 표시된다.  
  
별로 실용성을 느끼지 않는 사례지만 어떤 클래스의 일부의 거동을 오버라이드로 커스터마이즈 하고자 할 경우 계승 할 수밖에 없다.  
이런 경우는 원래 public 계승이 좋지만 어떤 이유(기저 클래스의 생성자가 virtual이 되지 못한 등)로 기저 클래스의 포인터에 의한 관리를 허락하지 않는 설계로 하고 싶은 경우는 전술대로 public계승하지 않기로 구현될 수 있다.  
  
STL의 컨테이너는 모두 생성자가 virtual가 아니다. 그래서 이들 클래스를 확장한 편리한 컨테이너 클래스를 만들고 싶은 경우도 절대로 public 계승해서는 안 된다. 하지만 예를 들어 std::vector의 확장 클래스를 만들고 싶을 때 std::vector를 멤버로서 가지면 std::vector가 가진 모든 메소를 재 구현하지 않으면 안 된다는 골치 아픈 일이 발생한다.  
  
이런 경우 계승과 using을 조합함으로써 구현을 간단하게 할 수 있다.  
```
template<typename T>
class MyVector
  :private std::vector<T, std;allocator<T>>{ //STL 컨테이너는 절대 public 계승해서는 안 된다
public:
  using std::vector<T>::push_back; //using으로 push_back의 접근 수준을 public로 올린다
};
 
int main()
{
  MyVector<int>v;
  v.push_back(99); // private 계승한 vector의 push_back에 접근할 수 있다.
  return 0;
}
```  
  
boost::noncopyable은 데이터를 가지지 않는 클래스이므로 EBO가 통하고, 또한 소멸자가 virtual이 아니라서 private 계승에 안성맞춤인 예이다.  
  
  
## 정리
1. public 계승 외는 기저 클래스의 포인터에 대입할 수 없다  
2. 소멸자가 virtual이 아닌 클래스는 절대 public 계승해서는 안 된다  
3. 계승이 아니라 내포로 구현될 때는 가급적 그렇게 하는 것이 좋다  
4. 비 public 계승해야 할 때는 특수가 존재하므로 c++의 거동을 제대로 이해하고 판단한다    
  