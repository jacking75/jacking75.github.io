---
layout: post
title: C++ - 자주 사용하는 C++ 이디엄(idiom) Extension Member Function
published: true
categories: [C++]
tags: c++ idiom Extension
---
C#의 확장 메소드와 같이 기존 클래스에 대해서 멤버 함수의 확장을 operator 오버로드를 구사하여 시뮬레이트 하는 idiom 이다.  
예컨대 std::vector에 print 라는 모든 요소를 출력하는 확장 구성원을 만들려고 한다면  
  
```
const struct Listed_print{
    using result_type=void;

    template<class Range>
    void operator()(const Range& r)
    {
        std::copy(r.cbegin(),r.cend(),std::experimental::make_ostream_joiner(std::cout,","));
    }
}listed_print={};

template<class Range,class F>
typename std::result_of<F(Range&)>::type operator|(const Range& r,F function)
{
    return function(r);
}


int main()
{
    std::vector<int> v{1,2,3,4,5};
    v | listed_print;
}
```  
  
  
반환 값이 필요한 경우 시그너처에서 argument 부분을 추출하여 result_of메타 함수로 반환 타입을 빼낸다.  
```
const struct Listed_print{
    template<class Range>
    const Range& operator()(const Range& r)const
    {
        std::copy(r.cbegin(),r.cend(),std::experimental::make_ostream_joiner(std::cout,","));
        std::cout<<std::endl;
        return r;
    }
}listed_print={};

template<class Range,class F>
auto operator|(const Range& r,F function)
{
    return function(r);
}
```  
  
  
반복자를 만들 때 보다 효율적으로 조작이 가능하다.   
```
static const auto reverse=[](auto& r){
    std::reverse(r.begin(),r.end());
    return r;
};
static const auto print=[](const auto& r){
    std::copy(r.cbegin(),r.cend(),std::experimental::make_ostream_joiner(std::cout," "));
    return r;
};
template<class Range,class Functor>
auto operator|(Range& r,const Functor& func)
{
    return func(r);
}
int main()
{
    std::vector<int> v{1,2,3,4,5};
    v | reverse | print;
}
```
  
  