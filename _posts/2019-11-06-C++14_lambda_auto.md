---
layout: post
title: C++14 - 람다식의 인수에 가변 길이 인수 사용하는 예제
published: true
categories: [C++]
tags: c++ c++14 lambda
---
[출처](http://developer.wonderpla.net/entry/blog/engineer/CPlusPlus11_Lambda/ )  
    
```
#include <iostream>  
using namespace std;  
  
void print (){  
     cout << endl;  
}  
  
template <typename T,typename ...Types>  
void print (T head, Types... tail) {  
    cout << head << endl;  
    print(tail...);  
}  
  
void test06() {  
    // auto 타입으로 가인수를 설정
    auto add = [](auto x){return x + x;};  
  
    // 「8」  
    cout << add(4) << endl;  
  
    // 「6.4」  
    cout << add(3.2) << endl;  
  
    // 「abcabc」  
    cout << add(std::string("abc")) << endl;  
  
    //가변 길이 인수도 가능
    //가변 길이 인수는 C++11의 가변 길이 템플릿이 사용된다
    auto cat = [](auto head,auto...tail) {  
        //파라메터 팩을 재귀적으로 전개
        print(head,tail...);  
    };  
  
    // 「1」  
    // 「3」  
    // 「4」  
    // 「nya」  
    // 「2.12」  
    // 「nya-」  
    cat(1,3,4,"nya",2.12,"nya-");  
  
    // 파라메터 팩을 전개
    auto cat02 = [](auto ... nya) {  
        for (auto nya_ : {nya...}) {  
            cout << nya_;  
        }  
    };  
  
    // 「nyanyannya-」  
    cat02("nya","nyan","nya-");  
}  
```  
  
