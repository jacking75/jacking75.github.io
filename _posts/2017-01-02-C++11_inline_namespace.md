---
layout: post
title: C++11 - 인라인 이름 공간(inline namespace)
published: true
categories: [C++]
tags: c++ c++11
---
### 개요
이름 공간 내에 있는 기능을 투과해서 접근할 수 있는 기능이다. 
즉 이름 공간(my_namespace) 안에 또 다른 이름 공간(features)을 inline로 선언하고 있으면 features에 정의된 기능을 my_namespace를 통해서 접근 할 수 있다.   

```
#include <iostream>

namespace my_namespace {
  inline namespace features {
    void f() { 
        std::cout << "call f()" << std::endl;
    }
  }
}

int main()
{
  my_namespace::features::f();
  my_namespace::f();           // features 이름 공간을 생략했다.
}
```  
  
<br> 
<br>  
  
  
### 문법

- 이름 공간의 inline 지정은 이름 있는 이름 공간과 이름 없는 이름 공간 양쪽에서 사용할 수 있다.
- inline 지정된 이름 공간을 "인라인 이름 공간(inline namespace)"이라고 부른다.  
  
```
inline namespace my_namespace {}
inline namespace {}
```  
  
- 인라인 이름 공간의 멤버는(이름 공간 안에 정의한 것) 이것의 외부 이름 공간의 구성원으로 사용할 수 있다.
- 인라인 이름 공간과 외부 이름 공간은 인수 종속의 이름 검색에서 검색되는 "관련 있는 이름 공간(associated namespace)" 이 된다

```
#include <iostream>

namespace ns1 {
  class X {};

  inline namespace ns2 {
	class Y {};

	void f(X)
	{
	  std::cout << "call f()" << std::endl;
	}
  }

  void g(Y)
  {
	std::cout << "call g()" << std::endl;
  }
}

int main()
{
  f(ns1::X());      // 「call f()」출력
  g(ns1::Y());      // 「call g()」출력
  g(ns1::ns2::Y()); // 「call g()」출력
}
```

- 인라인 이름 공간의 외부 이름 공간을 using 지시문을 지정하여 인라인 이름 공간의 구성원이 외부 이름 공간 멤버로 암시 적으로 삽입된다

```
#include <iostream>

namespace ns1 {
  inline namespace ns2 {
	void f()
	{
	  std::cout << "call f()" << std::endl;
	}
  }
}

int main()
{
  using namespace ns1;
  f();
}
```
  
- 인라인 이름 공간의 멤버는 외부 이름 공간에서 외부 이름 공간의 멤버 인 것처럼 명시적으로 인스턴스화 및 명시적 특수화 수 있다.

```
#include <iostream>

namespace ns1 {
  inline namespace ns2 {
	template <class T>
	struct X {
	  static constexpr int value = 0;
	};
  }

  // 인라인 이름 공간에서 정의된 클래스 템플릿을 명시적으로 인스턴스화
  template struct X<int>;

  // 인라인 이름 공간에서 정의된 클래스트 템플릿을 명시적으로 특수화
  template <>
  struct X<void> {
	static constexpr int value = 1;
  };
}

int main()
{
  std::cout << ns1::X<int>::value << std::endl;       // 0 출력
  std::cout << ns1::X<void>::value << std::endl;      // 1 출력
  std::cout << ns1::ns2::X<void>::value << std::endl; // 1 출력
}
```
 	
- 인라인 이름 공간의 멤버와 외부 이름 공간의 멤버가 이름이 같은 경우 외부 이름 공간으로 이 멤버에 접근하면 이름 충돌로 에러가 된다.

```
namespace ns1 {
  inline namespace ns2 {
	int a;
  }
  int a;
}

int main()
{
  ns1::a = 0; // 이름 충돌로 에러
  //ns1::ns2::a = 0;  // 이것은 ok
}
```
  
  
<br> 
<br>   
  
  
### API 버저닝

- 인라인 이름 공간을 기본 버전으로 하고, 오래된 API를 원래의 이름 공간에 그대로 남겨 둘 수 있다.  
기본 버전을 전환 할 때 기본 버전에 이름 공간을 인라인 이름 공간로 변경하고 기본 버전 이외의 이름 공간를 비 인라인 이름 공간로 변경한다.  
따라서 바이너리 호환성을 유지하면서 버전 관리를 쉽게 할 수 있다.
  
```
#include <iostream>

namespace my_namespace {
  namespace v1 {
	void f()
	{
	  std::cout << "v1" << std::endl;
	}
  }

  inline namespace v2 {
	void f()
	{
	  std::cout << "v2" << std::endl;
	}
  }
}

int main()
{
  my_namespace::v1::f(); // 오래된 버전의 API를 호출.  v1 출력
  my_namespace::v2::f(); // 버전을 명시적으로 지정하여 API를 호출.  v2 출력
  my_namespace::f();     // 기본 버전의 API를 호출. v2 출력
}
```


<br>
<br>    

### 출처  
- https://cpprefjp.github.io/lang/cpp11/inline_namespaces.html  
