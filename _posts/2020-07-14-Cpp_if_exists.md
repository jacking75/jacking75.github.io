---
layout: post
title: C++ - __if_exists
published: true
categories: [C++]
tags: c++ __if_exists
---
[출처](http://srz-zumix.blogspot.kr/2013/12/visual-c-ifexists.html  )  
  
Visual C++ 전용 키워드 이다.  
```
__if_exists ( identifier ) { 
    statements
};
```  
- identifier에 전달한 식별자가 존재한다면 statements가 실행된다.
- 부정형으로 `__if_not_exists`가 있다.
- 컴파일 타임 때 실행.
  
<br>  
  
```
template<typename T>
class X : public T {
public:
    void Dump() {
        __if_exists(T::Dump) {
            T::Dump();
        }
        __if_not_exists(T::Dump) {
            ::std::cout << "T::Dump does not exist" << ::std::endl;
        }
    }
};
 
class A {
public:
    void Dump() {
        ::std::cout << "A::Dump()" << ::std::endl;
    }
};
 
class B {};

int main() {
    X<A> x1;
    X<B> x2;
 
    x1.Dump();
    x2.Dump();
 
    return 0;
}
```  
  
  
```
class A {
public:
    static float get() { return 1.2f;  }
};
class B {};
 
int main() {
    auto a = __if_exists(A::get) {
        A::get();
    }
    __if_not_exists(A::get) {
        "not found";
    }
 
    auto b = __if_exists(B::get) {
        B::get();
    }
    __if_not_exists(B::get) {
        "not found";
    }
 
    auto c = [](){ 
        __if_exists(A::get) { return A::get(); }
        __if_not_exists(A::get) { return "not found"; }
    }();
 
    ::std::cout << a << ::std::endl;
    ::std::cout << b << ::std::endl;
    ::std::cout << c << ::std::endl;
}
```
  
  
```
int main()
{
    int x=0, y=1;
    auto fa = [__if_exists(x) { x } __if_not_exists(x) { y } ]()
    { 
        __if_exists(x) {
            ::std::cout << x << ::std::endl;
        }
        __if_not_exists(x) {
            ::std::cout << y << ::std::endl;
        }
    };
    auto fb =[__if_exists(z) { z } __if_not_exists(z) { y }]()
    {
        __if_exists(z) {
            ::std::cout << z << ::std::endl;
        }
        __if_not_exists(z) {
            ::std::cout << y << ::std::endl;
        }
    };
 
    fa();
    fb();
}
```
  
  
```
class A {
public:
    static void Dump() { ::std::cout << "A::Dump" << ::std::endl; }
};
class B {
public:
    static void Dump() { ::std::cout << "B::Dump" << ::std::endl; }
};
 
#define TEST1(name, base) class name : \
    public __if_exists(base) { base } __if_not_exists(base) { A } {};
#define TEST2(name, base) class name : \
    public decltype( __if_exists(base) { base() } __if_not_exists(base) { A() }) {};
 
TEST1(X1, B);
TEST1(Y1, C);
TEST2(X2, B);
TEST2(Y2, C);
 
int main()
{
    X1::Dump();
    Y1::Dump();
    X2::Dump();
    Y2::Dump();
 
    return 0;
} 
```  
  
  
```
class Test1 {
public:
    static float Dump() {
        ::std::cout << "Test1::Dump" << ::std::endl;
        return 0.1f;
    }
};
class Test2 {
public:
    static int  Print() {
        ::std::cout << "Test2::Print" << ::std::endl;
        return 1;
    }
};
 
 
template<typename T>
auto F(void) ->
    decltype( __if_exists(T::Dump) { T::Dump() }
        __if_exists(T::Print) { T::Print() } )
{
    __if_exists(T::Dump ) { return T::Dump();  }
    __if_exists(T::Print) { return T::Print(); }
}
 
int main(int argc, const char* argv[])
{
    auto var1 = F<Test1>();
    auto var2 = F<Test2>();
 
    ::std::cout << var1 << ::std::endl;
    ::std::cout << var2 << ::std::endl;
 
    return 0;
} 
```
  
  
```
class A {
public:
    ~A() = delete;
    void F() = delete; // delete 한 것도 __if_exists에서는 true가 된다.
};
 
int main()
{
    __if_exists(A::~A) {
        ::std::cout << "A::~A found" << ::std::endl;
    }
    __if_not_exists(A::~A) {
        ::std::cout << "A::~A not found" << ::std::endl;
    }
 
    __if_exists(A::F) {
        ::std::cout << "A::F found" << ::std::endl;
    }
    __if_not_exists(A::F) {
        ::std::cout << "A::F not found" << ::std::endl;
    }
 
    __if_exists(A::G) {
        ::std::cout << "A::G found" << ::std::endl;
    }
    __if_not_exists(A::G) {
        ::std::cout << "A::G not found" << ::std::endl;
    }
 
    return 0;
}
```
  
  
```
// 비 template한 형태의 이름이 아니면 안 된 것 같다.
// typedef와 형태가 결정적인 using의 경우는 제대로 판별할 수 있을 듯.
// __if_exists(A<int, int>::t)이라고 쓰면 좋을듯

template<typename T, typename U>
class A {
    T t;
    U u;
};
 
template<typename U>
using B = A<int, U>;
using C = A<int, int>;
typedef A<int, int> D;
 
#define CHECK(name)     \
    __if_exists(name) { \
        ::std::cout << #name " found" << ::std::endl    \
    }                   \
    __if_not_exists(name) {\
        ::std::cout << #name " not found" << ::std::endl    \
    }
 
 
int main()
{
    CHECK(A);
    CHECK(B);
    CHECK(C);
    CHECK(D);
    CHECK(A::t);
    CHECK(B::t);
    CHECK(B::u);
    CHECK(C::t);
    CHECK(D::t);
 
    __if_exists(B<float>) {
        ::std::cout << "B<float> found" << ::std::endl;
    }
    __if_not_exists(B<float>) {
        ::std::cout << "B<float> not found" << ::std::endl;
    }
 
    __if_exists(A<int, int>::t) {
        ::std::cout << "A<int, int>::t found" << ::std::endl;
    }
    __if_not_exists(A<int, int>::t) {
        ::std::cout << "A<int, int>::t not found" << ::std::endl;
    }
 
    return 0;
} 
```
  
  
```
static const int a=1;
 
constexpr int fa(void)
{
    __if_exists(a) {
        return a;
    }
    __if_not_exists(a) {
        return 0;
    }
}
constexpr int fb(void)
{
    __if_exists(b) {
        return b;
    }
    __if_not_exists(b) {
        return 0;
    }
}
 
int main() {
    ::std::cout << fa() << ::std::endl;
    ::std::cout << fb() << ::std::endl;
    return 0;
}
```
  