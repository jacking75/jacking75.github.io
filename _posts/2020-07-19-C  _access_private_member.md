---
layout: post
title: C++ - PRIVATE 멤버에 합법적으로 접근하기
published: true
categories: [C++]
tags: c++
---
[출처](http://cpp.aquariuscode.com/legal-private-access )  
  
```
class Employer{
float time_for_game_;//게임 시간
float time_for_anime_;//애니메이션의 시간
float time_for_family_;//가족과의 시간
public:
float time_for_work_;//일의 시간
 
// 인사
template<class Rank>
void greet(Rank target){
std;cout<<"hello"<<std::endl;
    }
};

//사장에게만은 인사 말고 휴가를 조르다
template<>
void Employer::greet(President target){
std;cout<<"holiday please"<<std::endl;
}

int main() {
   President takeda;
   Employer mami;
   Employer jun;
    
   jun.greet(mami);
   jun.greet(takeda);
   return 0;   
}
```  
  
외부에서 이 Employer의 private 멤버에 접근하여고 게임 시간을 몽땅 일 시간으로 옮기는 것은 가능할까?  
```
//Employer가 정의되어 있는 파일을 인클루드한다
#include"employer.hpp"
 
//↓ ↓ 여기부터가 핵심
namespace{
    class X{};
}

template <>
void Employer::greet(X target) {
    time_for_work_+=time_for_game_;  //게임 시간을 일하는 시간에 보태다
    time_for_game_=0;  //게임의 시간은 제로다!
}
//↑ ↑ 여기까지가 핵심
 
 
int main()
{
    //게임을 위한 시간을 표시
    Employer jun;
    jun.printTime();
    jun.greet(X(); //특수화한 greet을 부른다
    jun.printTime();

    return 0;
}
```
  
멤버에 템플릿 함수가 있으면 외부에서 이 함수의 특수화를 실시할 수 있다.  
이것을 이용하여 본래 불가침인 private 멤버의 내용을 마음대로 쓸 수 있다.  
  