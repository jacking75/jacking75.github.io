---
layout: post
title: C++ - std::map과 함수 포인터
published: true
categories: [C++]
tags: c++ stl map function pointer
---
예제 코드  
  
```
#include <iostream>
#include <map>
#include <string>

double Add(double a, double b){ return a + b; }
double Sub(double a, double b){ return a - b; }
double Mul(double a, double b){ return a * b; }
double Div(double a, double b){ return a / b; }

std::map<std::string , double(*)(double, double)> funcMap= {
    { "+", Add },
    { "-", Sub },
    { "*", Mul },
    { "/", Div }
};

int main(int argc, char* argv[]){
    while(true){
        std::string letter;
        std::cout << “연산자를 지정해 주세요 > " << std::flush;
        std::cin >> letter;

        if (funcMap.find(letter) != funcMap.end()){
            double     a, b;
            std::cout << "２개의 수치를 입력해 주세요 > " << std::flush;
            std::cin >> a >> b;
            std::cout << a << letter << b << " = ";
            std::cout << funcMap[letter](a, b) << std::endl;
        }
        else{
            std::cout << ＂연산자가 없습니다" << std::endl;
        }
    }
    return 0;
}
```  
  
