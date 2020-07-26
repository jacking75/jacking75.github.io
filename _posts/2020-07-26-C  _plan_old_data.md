---
layout: post
title: C++ - PLAIN OLD DATA
published: true
categories: [C++]
tags: c++ pod
---
[출처](http://cpp.aquariuscode.com/plain-old-data  )  

C++의 POD(Plain Old Data)는 C언어의 데이터와 호환을 가진 데이터 구조이다.  
  
memcpy로 데이터를 복사하거나 하지만 그것은 본질적인 것이 아니지만 어쨌든 POD의 가장 큰 의미는 C 언어의 데이터와 bit 수준으로 완전히 호환을 갖는다는 것이다.   
C++과 C 언어의 프로그램이 있을 경우 교환하는 데이터는 POD가 아니면 안 된다.  
  
  
## POD형의 정의
- 표준 레이아웃형(standard layout type)이다
- trivial type 이다
  
이 2개의 정의를 충족하면 그것은 틀림없이 POD 타입이다.  
간단하네!  
각각의 정의를 더욱 파고들어 가자.  
  
  
## 표준 레이아웃 타입
- virtual 함수를 가지지 않는다
- virtual 계승을 하지 않는다
- 참조 구성원이 없다
- 비 static 멤버에 대해서 복수의 접근 지정자를 가지지 않는다(중요!)
- 표준 레이아웃이 없고 타입을 계승하지 않는다
- 표준 레이아웃이 아닌 static 멤버를 가지지 않는다
- 계승하고 있어도 좋지만 비 static 멤버를 가진 클래스는 계승 트리에서 1개 밖에 없다.
  
4번째 항목에 주목!!!  
클래스의 부정한 메모리 레이아웃에서도 언급한 대로 복수의 접근 지정자로 멤버를 정의하고 있을 경우 표준 레이아웃 보증은 없다.  
  
  
## trivial type
• 보잘 것 없는 디폴트 생성자를 갖는다
• 보잘 것 없는 복제 동작 및 무브 동작을 한다
  
<br>  
  
많이 등장하는 "보잘 것 없는 ○○“ 가 궁금할 것이다.  
보잘 것 없는 기본 생성자를 갖고, 보잘 것 없는 복제 동작, 무브 동작을 한다는 것은 대충 말하면 컴파일러 정의의 이들의 동작을 한다는 것이다.  
예를 들면 복사 생성자를 사용자가 정의하면 이건 보잘 것 없는 타입이 아니다.  
인수가 없는 생성자를 사용자가 정의하면 컴파일러 정의의 기본 생성자사라지므로 보잘 것 없는 형이 아니다.  
   
자, 이제 자기가 쓴 클래스가 POD 인지 여부를 판단할 수 있게 되었다.  
하지만 이런 것은 잊어도 좋다.  
어떤 클래스가 POD인지 정말 알고 싶다면 std::is_pod을 사용한다.  
  
```
#include<iostream>
 
class YourType{
// 정의 상세
};
 
int main(){
  if(std::is_pod<YourType>::value) { // is_pod를 사용하여 POD 판정
    std::cout<<"POD다, C와 호환이 있어"<<std::endl;
  }else{
    std::cout<<"POD가 아니냐!절대로 C에 넘기지 말아!"<<std::endl;
  }
}
```  
  
  
## 그다지 틀리지 않는 POD 타입
- 스스로 생성자를 정의하지 않는다
- virtual 이라는 문자열을 치지 않는다
- 계승하지 않는다
- 멤버 변수는 private(혹은 public)밖에 정의하지 않는다
- 내장 타입(int, char, float 등)만 멤버로 가진다.
  
일단 이를 유지하면 대개는 POD이다.  
그래도 POD가 아닌 데이터 구조를 정의한다면, 거꾸로 대단하네? 라고 생각하면 된다 ^^  
  
아래는 모두 POD 이다.  
```
struct Color {
    unsigned char red_;
    unsigned char green_;
    unsigned char blue_;
};
```  
  
```
class Vec2{
  float x_;
  float y_;
 
public:
  Vec2()=default;
  ~Vec2()=default;
 
public:
  Vec2 operator+(const Vec2&rhs) const
  {
    Vec2 v;
    vx_=x_+rhs.x_;
    vy_=y_+rhs.y_;
    return v;
  }
 
  Vec2 operator-(const Vec2&rhs) const
  {
    Vec2 v;
    vx_=x_-rhs.x_;
    vy_=y_-rhs.y_;
    return v;
  }
    
  Vec2 operator*(const Vec2&rhs) const
  {
    Vec2 v;
    vx_=x_*rhs.x_;
    vy_=y_*rhs.y_;
    return v;
  }
    
  Vec2 operator/(const Vec2&rhs) const
  {
    Vec2 v;
    vx_=x_/rhs.x_;
    vy_=y_/rhs.y_;
    return v;
  }
};
```  
  
덤으로 POD가 아닌 타입의 static 멤버를 가지고 있는 것도 그 클래스의 PDO 성에는 영향이 없다.  
  
  
  
## 정리
1. POD는 C 언어 호환 데이터 구조
2. POD의 정의는 세심하게 정해진다
3. c++11 에서는 std;is_pod<T>::value로 판정한다  
