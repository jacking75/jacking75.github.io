---
layout: post
title: C++ - 컴파일러 최적화 copy elision
published: true
categories: [C++]
tags: c++ 컴파일러 최적화 copy_elision 
---
[c++ 생성자와 move constructor 관련해서 궁금한 점이 있습니다.](https://throwbug.com/281/c%2B%2B-%EC%83%9D%EC%84%B1%EC%9E%90%EC%99%80-move-constructor-%EA%B4%80%EB%A0%A8%ED%95%B4%EC%84%9C-%EA%B6%81%EA%B8%88%ED%95%9C-%EC%A0%90%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4)  
라는 질문을 보고 알게된 것이다.  

처음 질문을 봤을 때는 별거 아니라고 생각했는데 좀 더 보니 예전에 내가 C++11 표준 관련 강연을 했을 사용한 코드와 비슷한데 그때와 다른 결과가 나와서 구글링을 해보았다.  
질문의 코드를 아래와 같다(테스트를 위해 내가 약간 변경한 부분이 있다).  
```
#include <iostream>

class ABC
{
	int *m_N;
	int m_size;
public:
	ABC()
	{
		m_N = new int[100];
		m_size = 100;
        printf("1\n");
	}
	ABC(int nSize) {
		m_N = new int[nSize];
		m_size = nSize;
		printf("2\n");
	}
	~ABC()
	{
		delete[] m_N;
	}
	ABC(const ABC& rhs)
	{
		m_size = rhs.m_size;
		m_N = new int[m_size];
		//memcpy(m_N, rhs.m_N, sizeof(int)*m_size);
        printf("3\n");
	}
	ABC(ABC&& rhs)
	{
		m_size = rhs.m_size;
		m_N = rhs.m_N;
		rhs.m_N = nullptr;
        printf("4\n");
	}
};

int main()
{
	ABC aa = ABC(10000);
    return 0;
}
```  

예전에 내가 위와 같은 코드를 실행했을 때는 2와 4가 출력되었는데 최신 버전의 컴파일러에서 해보면 2만 출력한다.  
사실 코드를 보면 2만 출력하는 것이 똑똑한 행동이다.  
그럼 왜 예전과 달라졌지 라고 알아보니 C++11에서 **copy elision** 라는 것이 C++ 스펙에 추가되었다.  
[copy elision](https://en.cppreference.com/w/cpp/language/copy_elision )
이것은 컴파일러가 불필요한 생성자 호출을 제외시켜 주는 컴파일러 최적화 옵션이다.  
컴파일러 모두를 다 검사해보지 않았지만 GCC는 버전 4 이후부터는 지원하고, Visual Studio 2017도 지원하고 있다.  

만약 copy elision 최적화를 off 시키고 싶다면 GCC의 경우 컴파일러 옵션으로 아래 항목을 추가한다.  
```
-fno-elide-constructors
```  
  
위 코드에서 **-fno-elide-constructors** 옵션을 추가하고 컴파일하면 4와 2가 출력된다.  
[테스트 코드](https://wandbox.org/permlink/Dqss90YKH6OGlJo9 )   
  
<br/>  
<br/>
  
  
# Copy elision
[위키피디아](https://en.wikipedia.org/wiki/Copy_elision )  

C++ 프로그래밍에서 복사 생략(copy elision)은 불필요한 객체의 복사를 제거하는 컴파일러 최적화 기술이다.  
C++ 표준은 기본적으로 완성된 프로그램의 표면상의 동작이 언어 표준이 요구 한대로라면 구현은 어떤 최적화를 해도 좋다고하고있다.( as-if 규칙 )  


그러나 프로그램의 동작이 변경 될 수 있음에도 불구하고, 여전히 복사가 생략 되는 상황을 언어 표준은 몇 가지 들고있다.  
그 중 가장 유명한 것이 반환 값 최적화이며, 다른 하나는 클래스 타입의 임시 개체가 동일한 타입의 개체로 복사 할 때의 최적화이다.  
이것도 널리 구현 되어 있으며, C++ 표준에서 거론 되고 있다.  
결과적으로 복사 초기화와 직접 초기화 사이에 성능 차이는 없지만, 의미상으로는 그렇지 않고, 복사 초기화는 접근 가능한 복사 생성자가 여전히 필요하다.  
또한 참조에 속박되는 임시 객체는 최적화 되지 않는다. 예:  
```
#include <iostream>
int n = 0;
struct C {
  explicit C(int) {}
  C(const C&) { ++n; } // 눈에 보이는 부작용을 가진 복사 생성자
};                     // 정적인 기억공간을 가진 오브젝트를 변경한다

int main() {
  C c1(42); // 직접 초기화. C::C(42) 를 호출
  C c2 = C(42); // 복사 초기화. C::C( C(42) ) 를 호출
  
  std::cout << n << std::endl; // 복사가 생략 되었다면 0, 그렇지 않으면 1을 출력한다.
  return 0;
}
```
  
언어 표준에 따르면 예외로 던져지는 개체도 같은 최적화가 적용된다.  
그러나 던져진 객체에서 예외 개체에 복사하고, 예외 객체에서 예외 선언의 catch 절에 복사 모두 최적화 될 수고 있고 되지 않을 수도 있다.  
이것은 임시 개체에도 이름이 있는 객체에서도 마찬가지이다. 예:  
```
#include <iostream>

struct C {
  C() {}
  C(const C&) { std::cout << "Hello World!\n"; }
};

void f() {
  C c;
  throw c; // 이름 있는 오브젝트 c 를 예외 오브젝트에 복사.
}          // 이 복사가 생략될지 어떨지는 불확실하다.

int main() {
  try {
    f();
  }
  catch(C c) {  // 예외 오브젝트를 예외 선언 내의 임시 영역에 복사.
  }             // 이; 복사도 생략될지 어떨지는 불확실하다.
}
```
  

언어 표준을 준수하는 컴파일러는 위의 코드에서 "Hello World!"라고 두번 출력하는 프로그램을 생성해야했다.  
현재 버전의 C++ 표준(C++11)는 이름 있는 객체의 예외 객체에 복사 및 예외 핸들러에서 선언된 객체에 복사를 생략하는 것을 근본적으로 허용하는 것으로 이 문제는 해결되었다.  

GCC는 **-fno-elide-constructors** 옵션을 제공하여 복사 생략을 무효화 할 수 있도록 하고있다.  
이 옵션은 반환 값 최적화의 효과를 발견(또는 놓친!)하는데 유용하다.  
보통은 이 중요한 최적화를 비활성화 하는 것은 권장하지 않는다.  
