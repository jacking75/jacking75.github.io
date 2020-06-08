---
layout: post
title: C++ - std::bind 사용
published: true
categories: [C++]
tags: c++ stl bind
---
지정한 함수를 wrap해서 std::function을 만든다.  
  
```
#include <iostream>

void test_function(int a, int b) {
    printf("a=%d, b=%d\n", a, b);
}

int main(int argc, const char * argv[]) {
    auto func1 = std::bind(test_function, std::placeholders::_1, std::placeholders::_2);
    func1(1, 2);
    // -> a=1, b=2

    auto func2 = std::bind(test_function, std::placeholders::_1, 9);
    func2(1);
    // -> a=1, b=9

    return 0;
}
```  
  
std::placeholders::_n 이라는 것이 헷갈리지만 이것은 만든 std::function을 호출할 때의 인수를 가리킨다.
  
<br>  
  
```
#include <iostream>

class Test {
public:
    void test_function(int a, int b) {
        printf("a=%d, b=%d\n", a, b);
    }
};

int main(int argc, const char * argv[]) {
    auto test = Test();
    auto func1 = std::bind(&Test::test_function, test, std::placeholders::_1, std::placeholders::_2);
    func1(1, 2);

    return 0;
}
```  
  
<br>  
  
```
#include <iostream>

class Test {
public:
    void test_function(int a, int b) {
        printf("a=%d, b=%d\n", a, b);
    }

    void bind_test() {
        auto func1 = std::bind(&Test::test_function, this, std::placeholders::_1, std::placeholders::_2);
        func1(1, 2);
    }
};

int main(int argc, const char * argv[]) {

    auto test = Test();
    test.bind_test();

    return 0;
}
```  
  
    