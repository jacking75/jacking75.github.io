---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) CRTP
published: true
categories: [C++]
tags: c++ idiom CRTP
---
CRTP는 Curiously Recurring Template Pattern의 약어로 자주 직역되고 있지만 기묘하게 재기한 템플릿 패턴이라고 부른다.  
```
template<class Inher>

struct X{};

struct Derived:X<Derived>{}; // CRTP
```  
  
  
기본 클래스에서 파생 클래스를 볼 수 있으므로 이것을 이용한 다양한 패턴도 다시 생겨나고 있다.  
예를 들면 순수 가상 함수 등을 이용하지 않고 정적으로 파생 클래스에 대해서 해당 구현을 강요하는 등. 
```
template<class Tp>
struct base{
    void f()const{static_cast<Tp>(this)->f();}
};
struct derived:private base<derived>{
    void f() const {}
};
```  
  
  
그러나 이 기술만으로는 파생 클래스에서는 구현을 하지 않고도 멤버를 호출하지 않으면 오류를 내지 않으므로 아래처럼 한다.  
```
template<class Tp,void (Tp::*)()>
struct signature_f_1{using type=Tp;};

template<class Tp,class=Tp>
struct signature_f_2:std::false_type{};
template<class Tp>
struct signature_f_2<Tp,typename signature_f_1<Tp,&Tp::f>::type>:std::true_type{};

template<class Tp>
constexpr bool signature_f_v=signature_f_2<Tp>::value;

template<class Derived>
struct base{
    virtual ~base()=default;
    base(){static_assert(signature_f_v<Derived>,"u must define function f");}
    void f(){return static_cast<Derived*>(this)->f();}
};
```  
```
template<class Tp>
class equal_comp{
    friend bool operator!=(const Tp& lhs,const Tp& rhs){return !lhs.operator==(rhs);}
};

class X final:equal_comp<X>{
    std::string s;
public:
    bool operator==(const X& rhs)const{return rhs.s==s;}
    explicit X(const std::string& s):s(s){}
};

int main()
{
    using namespace std::string_literals;
    X a("hoge"s),b("foo"s);
    assert(a!=b);
}
```  
  
`==` 와 `!=` 처럼 각각의 연산자가 쌍을 이루고 있는 연산자는 한쪽 구현이 확정된 순간에 당연히 다른 구현이 확립되므로 자동 생성이 가능하다.  
  
  
  
  
  
  
  