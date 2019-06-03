---
layout: post
title: C++ - 함수 핸들러(Funtion Dispatcher) (Code)
published: true
categories: [C++]
tags: c++ code Dispatcher
---
[온라인 서버 제작자 모임의 핵랑님이 만든 것](http://cafe.naver.com/ongameserver/2612 http://cafe.naver.com/ongameserver/2921)    
  비공개 카페라서 링크를 눌러도 멤버가 아니라면 접속할 수 없습니다.  
    
    
```
#pragma warning(disable:4819)
#include <boost/bind.hpp>
#include <boost/function.hpp>
#include <boost/lambda/lambda.hpp>
#pragma warning(default:4819)
#include <string>
#include <vector>
#include <utility>
#include <iostream>
 
namespace Dispatcher
{
    template<typename IdentifierT, typename FunctorT>
    class Mapper
    {
    private:
        typedef std::vector<std::pair<IdentifierT, FunctorT> > FunctorMap;
 
    private:
        FunctorMap functorMap_;
 
    public:
        bool Add(IdentifierT id, const FunctorT &functor)
        {
            for (FunctorMap::const_iterator it = functorMap_.begin();
                it != functorMap_.end(); ++it)
            {
                if ((*it).first == id)
                {
                    assert(!"id is already exists.");
                    return false;
                }
            }
 
            functorMap_.push_back(std::make_pair(id, functor));
 
            return true;
        }
 
        bool Remove(IdentifierT id)
        {
            bool result = false;
 
            for (FunctorMap::const_iterator it = functorMap_.begin();
                it != functorMap_.end(); ++it)
            {
                if ((*it).first == id)
                {
                    functorMap_.erase(it);
                    result = true;
                    break;
                }
            }
 
            return result;
        }
 
        bool DoDispatch(IdentifierT id) const
        {
            FunctorT functor;
            if (Find(id, functor) == true)
            {
                functor();
                return true;
            }
 
            return false;
        }
        template<typename Arg1T>
        bool DoDispatch(IdentifierT id, Arg1T arg1) const
        {
            FunctorT functor;
            if (Find(id, functor) == true)
            {
                functor(arg1);
                return true;
            }
 
            return false;
        }
        template<typename Arg1T, typename Arg2T>
        bool DoDispatch(IdentifierT id, Arg1T arg1, Arg2T arg2) const
        {
            FunctorT functor;
            if (Find(id, functor) == true)
            {
                functor(arg1, arg2);
                return true;
            }
 
            return false;
        }
 
    private:
        bool Find(IdentifierT id, FunctorT &item) const
        {
            bool result = false;
 
            for (FunctorMap::const_iterator it = functorMap_.begin();
                it != functorMap_.end(); ++it)
            {
                if ((*it).first == id)
                {
                    item = (*it).second;
                    result = true;
                    break;
                }
            }
 
            return result;
        }
    };
}
 
void Func(const std::string &s)
{
    std::cout << "Func: " << s << std::endl;
}
 
void Func(unsigned int n, const std::string &s)
{
    std::cout << "Func: " << n << ", " << s << std::endl;
}
 
struct Test
{
    void Func(unsigned int n, const std::string &s, unsigned int x)
    {
        std::cout << "Test::Func: "
            << n << ", " << s << ", " << x << std::endl;
    }
} 
 
using namespace boost;
 
function<void(const std::string &, const std::string &)> lambda_f = (
    std::cout << lambda::constant("lambda expr #1: ")
    << lambda::_1 << lambda::_2 << lambda::constant("\n"));
 

int main(void)
{
    Test test;
    
    // Dispatcher 설정
    Dispatcher::Mapper<
        unsigned int, function<void(const std::string &)> > mapper;
    mapper.Add(0, bind(&Func, _1));
    mapper.Add(1, bind(&Func, 10, _1));
    mapper.Add(2, bind(&Test::Func, test, 10, _1, 20));
    mapper.Add(3, (std::cout << lambda::constant("lambda expr #2: ")
        << lambda::_1 << lambda::constant("\n")));
    mapper.Add(4, bind(lambda_f, "Hello, ", _1));
 
    // Dispatcher 테스트
    std::string s = "test";
 
    // DoDispatch<std::string>()로 해석. 따라서 std::string 임시 객체 생성됨.
    mapper.DoDispatch(0, s);
 
    // 임시 객체 생성을 막기 위해 타입을 명시함
    mapper.DoDispatch<const std::string &>(1, s);
    mapper.DoDispatch<const std::string &>(2, s);
    mapper.DoDispatch<const std::string &>(3, s);
    mapper.DoDispatch<const std::string &>(4, "World!");
 
    return 0;
}
```
  