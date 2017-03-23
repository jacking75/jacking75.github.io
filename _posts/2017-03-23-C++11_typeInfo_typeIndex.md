---
layout: post
title: C++11 - type_info와 type_index
published: true
categories: [C++]
tags: c++ c++11  type_info type_index
---
C++에서는 RTTI(Run Time Type Infomation/Identification)라고 불리는 기능이 있다.
( 타 언어의 리플렉션과 좀 비슷한데 기능은 훨씬 떨어짐 ^^;)
std::type_info는 타입 정보를 표현하는 클래스이다. type_info 객체는 typeid 식에서 얻을 수 있다.  
typeid 식은 오퍼랜드 식 혹은 type-id를 취한다.  
typeid 식의 결과는 const한 std::type_info 타입 lvalue 이다.  
typeid 식은 실행 시 타입 정보를 얻는다(컴파일 타임이 아니다!!!)
  
<br> 
<br>  

 
### 사용 방법  
typeid 식을 사용하려면 typeinfo 헤더를 include 할 필요가 있다.

```
#include <typeinfo>
```
  
<br> 
<br>  
  
    
### 예제 1

```
#include <iostream>       
#include <typeinfo>

int main()
{
    std::type_info const *ptr;
    {
        ptr = &typeid(int);
    }

    std::cout << ptr->name() << std::endl;
}
```
  
<br> 
<br>  
  
    
### 예제 2

```
// http://www.cplusplus.com/reference/typeinfo/type_info/name/
// type_info::name example
#include <iostream>       // std::cout
#include <typeinfo>       // operator typeid

int main() {
  int i;
  int * pi;
  std::cout << "int is: " << typeid(int).name() << '\n';
  std::cout << "  i is: " << typeid(i).name() << '\n';
  std::cout << " pi is: " << typeid(pi).name() << '\n';
  std::cout << "*pi is: " << typeid(*pi).name() << '\n';

  return 0;
}
```
  
결과  
<pre>
int is: int
  i is: int
 pi is: int *
*pi is: int
</pre>

  
<br> 
<br>  
  
### 예제 3  
typeid는 비교 연산을 할 수 있다.

```
typeid(int) == typeid(double) ; 
```
  
<br> 
<br>  
  
### 예제 4

```
#include <iostream>       
#include <typeinfo>

struct Base { virtual void p() {} };
struct Cat : Base { } ;
struct Dog : Base { } ;

std::string get_name( Base & ref )
{
    decltype(auto) ti = typeid(ref) ;
    if ( ti == typeid(Cat) )
        return "고양이" ;
    else if ( ti == typeid(Dog) )
        return "개" ;

    return "모름" ;
}

int main()
{
    Base base;
    std::cout << get_name(base) << std::endl;
    
    Cat cat;
    std::cout << get_name(cat) << std::endl;
    
    Dog dog;
    std::cout << get_name(dog) << std::endl;
    
    return 0;
}
```
  
**Base 클래스는 꼭 가상함수를 가지고 있어야** get_name 함수가 올바르게 동작한다.
  
  
<br> 
<br>  
  
### 예제 4

```
#include <iostream>
#include <map>
#include <typeindex>

int main()
{
    std::map< std::type_index, std::string > m =
    {
        { typeid(int), "int" },
        { typeid(double), "double"}
    } ;

    std::cout << m[typeid(int)] << '\n' ;
}
```
  
  
<br> 
<br>  
  
### 참고  
- https://cpplover.blogspot.kr/2014/10/ctypeidtypeinfotypeindex.html
- http://www.cplusplus.com/reference/typeinfo/type_info/

