---
layout: post
title: C++ - C++/CLI
published: true
categories: [C++]
tags: c++ cli net
---
## 클래스(class), 핸들(^)
- C++/CLI에서 관리 클래스는 ‘ref’라는 키워드를 사용하여 만든다.
- 비 관리 클래스
  
```
class Test
{
};
```
  
- 관리 클래스
  
```
ref class Test
{
};
```
  
  
관리 클래스는 ‘ref’를 제외하고는 외관상으로는 비 관리 클래스와 비슷하지만 관리와 비 관리가 많이 다르듯이 관리 클래스는 비 관리 클래스와 다른점이 있다. 그러므로 이것을 잘 파악하고 있어야 한다.  

- 관리 클래스의 특징
    - 정적 할당은 안 된다. 무조건 가비지컬렉션(GC)에 동적으로 생성한다.
    - 복사 생성자를 만들 수 없다.
    - ‘^’(핸들이라고 부른다)과 ‘gcnew’를 사용하여 클래스를 생성한다. 당근 메모리 해제는 GC에서 관리한다.
    - 핸들은 네이티브의 ‘*’(포인터) 나 ‘&’(참조)와 비슷한 것으로 더 안정스럽다.
    - ‘delete’는 명시적으로 클래스에서 사용하고 있는 리소스를 해제할 때 사용하는 것으로 ‘소멸자’가 호출 되는 것이지 메모리에서 해제하는 것은 아니다. 즉 delete로 GC에 있는 메모리를 해제하는 것은 아니다.
    - delete로 소멸자를 호출하면 GC에 의해서 진짜 파괴될 때 소멸자는 호출되지 않는다. (소멸자를 선언한 클래스는 자동적으로 IDisposable 인터페이스를 구현한다. 소멸자는 컴파일러에 의해서 Dispose() 메소드로 치환된다).
    - 소멸자 이외에 'finalize'를 선언할 수 있다. finalize는 GC에서 인스턴스가 파괴될 때 호출된다. 단 delete로 소멸자를 호출한 경우에는 finalize는 호출되지 않는다.
    - finalize는 ‘!’ 키워드를 사용한다.
  
- 관리 클래스 사용해 보기. < 관리 클래스 정의 및 사용 >
  
```
#include "stdafx.h"

using namespace System;

ref class ManagedTest
{
public:
		   ManagedTest() { Console::WriteLine(L"New ManagedTest"); }
		   ~ManagedTest() { Console::WriteLine(L"delete ManagedTest"); }
		   !ManagedTest() { Console::WriteLine(L"finalize ManagedTest"); }
};

int main(array<System::String ^> ^args)
{
		   Console::WriteLine(L"1.");
		  ManagedTest^ MTest1 = gcnew ManagedTest();
		   delete MTest1;

		   Console::WriteLine(L"2.");
		   ManagedTest^ MTest2 = gcnew ManagedTest();

		   return 0;
}
```
  
- gcnew로 생성하지 않기
    - C++/CLI는 클래스를 생성할 때 ‘gcnew’를 사용하지 않고 생성할 수도 있다.

  
<‘gcnew’를 사용하지 않고 클래스 생성하기 >
  
```  
#include "stdafx.h"
#include <stdio.h>

using namespace System;

ref class ManagedTest
{
public:
		   ManagedTest() { Console::WriteLine(L"New ManagedTest"); }
		   ~ManagedTest() { Console::WriteLine(L"delete ManagedTest"); }

		   void func() { Console::WriteLine(L"Call func() - {0}", nNumber ); }

		   int nNumber;
};

void foo1()
{
		   ManagedTest MTest;
		   MTest.nNumber = 1;
		   MTest.func();
}

void foo2()
{
		   ManagedTest^ MTest = gcnew ManagedTest();
		   MTest->nNumber = 2;
		   MTest->func();
}

int main(array<System::String ^> ^args)
{
		   foo1();
		   foo2();

		   getchar();
		   return 0;
}
```
  
<리스트 1>의 ManagedTest MTest; 는 비 관리 C++처럼 GC가 아닌 스택 영역에 생성하는 것으로 착각할 수도 있겠지만 전혀 아니다. 클래스를 생성하면 언제나 GC 영역에 만들어진다.  
위의 코드는 그냥 ‘gcnew’ 사용을 우리가 생략하고 컴파일러가 대신 써 준다고 생각하면 된다.  
  
```
void foo1()
{
		   ManagedTest MTest;
		   MTest.nNumber = 1;
		   MTest.func();
}
``` 
  
은 컴파일러에 의해서 아래의 코드로 바뀐다.  
```
void foo1()
{
	ManagedTest^ MTest = gcnew ManagedTest();
	MTest->nNumber = 1;
	MTest->func();
	delete MTest;
}
```
  
위의 코드를 보시면 알 수 있듯이 gcnew를 사용하지 않고 클래스를 생성하면 우리가 직접 delete를 쓰지 않아도 되는 편리함을 얻을 수 있다. gcnew를 사용하지 않는 것은 C#에서 ‘using’을 간략화 시킨 것으로 생각하면 좋다.  
  
< C#의 using 문 사용 예 >  

```
using (Graphics g = this.CreateGraphics())
{
	g.DrawLine(Pens.Black, new Point(0,0), new Point(3,5));
}
```
  
  
## value 클래스
- 관리 클래스는 복사 생성자와 대입 연사자를 가지지 못하므로 아래의 코드는 컴파일 에러가 발생한다.
  
```
#include "stdafx.h"
#include <stdio.h>

using namespace System;

ref class C {
	int i;
};

void func(C c) {}

int main()
{
	C c;
	C d;
	func(c);   // 에러
	d = c;     // 에러
}
```
  
그러나 클래스를  ‘ref’가 아닌 ‘value’ 클래스로 정의하면 위의 코드는 컴파일 할 수 있다.  
```
#include "stdafx.h"
#include <stdio.h>

using namespace System;

value class C {
	int i;
};
void func(C c) {}
int main()
{
	C c;
	C d;
	func(c);   // 에러
	d = c;     // 에러
}
```
  
value 클래스는 클래스간 복사를 할 수 있는 능력이 있다. 이 복사는 비트 단위의 복사로 이른바 ‘memcpy’와 같은 복사이다.  
value 클래스는 복사를 할 수 있으므로 value 클래스의 멤버는 절대 복사 가능한 멤버만 가질 수 있다. 그래서 아래와 같은 value 클래스 C는 컴파일 에러가 발생한다.  
  
```
ref class A
{
  int i;
};

value class C
{
  A a;
};
```
  
또 value 클래스의 특징으로는 ref 클래스가 GC에서 만들어지는 것과 달리 스택에 만들 수 있다.  
```
value class C
{
};

C c;
```
  
위 코드에서 C 클래스는 스택에 만들어진다(물론 gcnew를 사용하면 GC에 만들어진다).  
value 클래스의 특징은 좀 더 있는데 위에 설명한 것들과 포함해서 아래와 같이 정리할 수 있다.  

- value 클래스의 특징
    - 기본 생성자를 가질 수 없다.
    - 복사 생성자를 가질 수 없다.
    - 대입 연산자를 가질 수 없다.
    - 소멸자를 가질 수 없다.
    - finalize를 가질 수 없다.
    - 클래스간 복사를 할 수 있다.
    - 복사 불가능한 것을 멤버로 가질 수 없다(ref 클래스 등).
    - 스택에 생성할 수 있다.
    - 다른 클래스를 계승할 수 없다(단 interface는 가능하다)
  
  
 
## 관리 클래스를 파라메터로 넘기기

```
ref class A
{
		   int i;
};

void foo1( A a )
{
}
```
  
함수 foo1의 파라미터 정의는 클래스 A가 value 클래스일 때만 사용할 수 있다. 그러므로 아래와 같이 foo1의 파라미터를 정의해야 한다.  

```
void foo1( A^ a )
{
}
```
  
- C++/CLI에서 참조는 ‘%’을 사용한다. 이것은 비 관리의 ‘&’와 구별이 된다.
- %는 아래와 같은 경우에 유용하게 사용할 수 있다.
  
```
void foo( A^ a )
{
}

A a;
// foo( a );  // 에러
foo( %a ); // 성공
```
  
- 참조는 C#에서는 'ref', VB.NET에서는 'ByRef'와 같다고 생각하면 된다.
  
```
// 값을 참조로 넘기는 경우
void valuebyref(int%i)
{
   i=5;
}

// 참조형을 참조로 넘기기
void refbyref(String^%s)
{
   s="newstring";
}

void main()
{
   int i=1;

   valuebyref(i);

   String^s="basestring";
   refbyref(s);
}
```
  
(위 코드는 http://adversaria-june.blogspot.com/2006/08/ccli_26.html 에서 인용 )  
  
  
  
## 구조체
- C++/CLI에서의 구조체가 클래스와 다른 점
    - 'value class'와 같습니다. 물론 비 관리코드에서는 기존의 C++과 같다
    - 구조체는 상속 받을 수 없다.
  
  
## 관리 클래스 전방 선언
- A라는 클래스를 전방선언 하고 싶다면 아래와 같이한다.
  
```
ref class A;
```
   
  
## 관리와 비관리의 혼재
- ref Class에서 비관리 클래스 가지기
  
```
#include <string.h>
#include <stdlib.h>
using namespace System;

ref class REFCLASS
{
	void clearp()
	{
		if (p)
		{
			free((void*)p);
			p = NULL;
		}
	}

public:
	char* p;
	~REFCLASS()
	{
		clearp();
	}
	!REFCLASS()
	{
		clearp();
	}
};

int main(array<System::String ^> ^args)
{
	{
		REFCLASS c;
		c.p = strdup("AAA");
		Console::WriteLine(gcnew String(c.p));

		REFCLASS^ hc = gcnew REFCLASS;
		hc->p = strdup("BBB");
		Console::WriteLine(gcnew String(hc->p));
	}
	return 0;
}
```
  
- 비관리 클래스에서 관리 클래스 가지기
    - gcroot를 사용한다.
  
```
#include <gcroot.h>
using namespace System;

class NativeClass
{
public:
	gcroot<String^> clrstring;
};


int main(array<System::String ^> ^args)
{
	String^ s = L"AAA";
	NativeClass n;
	n.clrstring = s;

	Console::WriteLine(n.clrstring);
	return 0;
}
```
  
  
## 배열
- C++/CLI에서의 배열은 ‘array’
- 비관리 C++에서는 배열은 ‘[]’을 사용한다.
  
```
int Nums[10];
char szName[20] = {0,};
```
  
그러나 C++/CLI에서의 배열은 ‘array’라는 클래스를 사용한다.  
int 형의 3개의 요소를 가지는 배열은 아래와 같이 정의합니다.  
  
```
array< int >^ A1 = gcnew array< int >(3);
array< int >^ A2 = gcnew array< int >(4) { 1, 2, 3 };
array< int >^ A3 = gcnew array< int >{ 1, 2, 3 };
```
  
다음은 간단한 사용 예.  
```
int main()
{
   array< int >^ Numbers = gcnew array< int >(5);

   for( int i = 0; i < 5; ++i )
   {
			 Numbers[ i ] = i;

			 System::Console::WriteLine( Numbers[i] );
   }

   getchar();
   return 0;
}
``` 
  
  
- array에 유저 정의형 사용하기  
array에는 기본형(int, float 등)만이 아닌 유저 정의형도 사용할 수 있다. 다만 비관리 클래스는 안된다. 오직 관리 클래스(ref class)만 가능하다. 또 그냥 ref 클래스를 그대로 넣을 수는 없는 클래스의 핸들을 사용해야 한다(ref 클래스는 GC에 동적 할당을 하기 때문).  
  
```
ref class refTest
{
};

array< refTest >^ arrTest;    // 에러
array< refTest^ >^ arrTest;   // OK
```
  
- for each 사용하기  
앞서 예제에서는 배열의 모든 요소를 순환하기 하기 위해 ‘for’문을 사용했다. 그러나 .NET에서는 for문 보다 ‘for each’문을 사용하는 것이 성능이나 안정성 등에서 더 좋다.  
  
```
#include <iostream>


int main()
{
		   array< int >^ Numbers = gcnew array< int > { 10, 11, 12, 13, 14 };

		   for each( int nValue in Numbers )
		   {
					 System::Console::WriteLine( nValue );
		   }

		   getchar();
		   return 0;
}
```
  
- 다 차원 배열  
array는 너무 당연하게 다 차원 배열도 만들 수 있다.  
  
```
int main()
{
		   array<int,2>^ a2 = gcnew array<int,2>(4,4);
		   array<int,2>^ b2 = gcnew array<int,2>{ {1,2,3,4},{1,2,3,4},{1,2,3,4},{1,2,3,4} };

		   getchar();
		   return 0;
}
```
  
  
## nullptr, interior_ptr, pin_ptr
- nullptr  
C/C++에서 포인터를 초기화 할 때 ‘NULL’을 사용한다. 그러나 C++11에서는 포인터를 초기화 할 때 NULL 대신 새로 생긴 ‘nullptr’을 사용할 수 있게 되었다.   
- C++/CLI는 이전부터 nullptr이 있었다.
    - C++/CLI에서는 ref 클래스의 핸들을 초기화 할 때는 nullptr을 사용한다.
    - C++/CLI, C++11의 nullptr은 C/C++ 처럼 ‘0’이 아니라는 것을 잘 기억하시기 바란다.
- interior_ptr  
interior_ptr은 관리 힙(managed heap. 즉 GC) 상의 value type나 기본형을 가리키는 포인터라고 할 수 있다. interior_ptr는 value type나 기본형을 비관리 코드의 포인터처럼 사용하고 싶을 때 사용하면 좋다.  
  
```
ref class REFClass
{
public:
	int nValue;
};

void SetValue( int* nValue )
{
	*nValue = 100;
}


int main()
{
	REFClass^ refClass = gcnew REFClass;
	SetValue( &refClass->nValue );  // 에러
}
```
  
위 코드를 빌드 해 보면 SetValue( &refClass->nValue ); 에서 빌드 에러가 발생한다. 매니지드 힙에 있는 것은 그 위치가 변하므로 비 관리 코드의 포인터를 넘길 수가 없습니다. 그럼 <코드 1>를 정상적으로 빌드 하기 위해서 interior_ptr를 사용해 보겠다.  
  
```
ref class REFClass
{
public:
	int nValue;
};

void SetValue( interior_ptr<int> nValue )
{
	*nValue = 100;
}


int main()
{
	REFClass^ refClass = gcnew REFClass;
	SetValue( &refClass->nValue );
}
```  
  
위의 SetValue의 파라미터로 비관리 코드의 참조나 포인터를 넘길 수도 있다.  
  
```
#include <iostream>

void SetValue( interior_ptr<int> nValue )
{
	*nValue = 100;
}

int main()
{
		   int nValue = 50;
		   SetValue( &nValue );

		   std::cout << nValue << std::endl;

		   getchar();
		   return 0;
}
```
  
그리고 interior_ptr에 대신 C++/CLI의 참조(‘%’)를 사용하는 방법도 있다.  

- pin_ptr  
pin_ptr은 관리 힙 상의 value type나 기본형을 비관리 코드에서 포인터로 사용하고 싶을 때 사용하는 기능이다. 가장 필요한 경우가 C++/CLI에서 기존의 비관리 코드로 만들어 놓은 라이브러리를 사용할 때이다.  
  
```
ref class REFClass
{
public:
	int nValue;
};

void SetValue( int* pValue )
{
	*pValue = 100;
}


int main()
{
	REFClass^ refClass = gcnew REFClass;
	pin_ptr<int> pValue = &refClass->nValue;
	SetValue( pValue );
	pValue = nullptr;
}
```  
  
pin_ptr에 메모리 주소를 할당하는 것을 ‘pin’이라고 부르고 사용이 끝난 후 nullptr로 초기화 하는 것을 ‘unpin’ 이라고 부른다. pin_ptr 사용이 끝난 후 가능한 빨리 unpin 해주는 것이 좋습니다.  

- interior_ptr과 pin_ptr의 차이점
    - interipor_ptr과 pin_ptr은 둘 다 관리 힙 상의 value type이나 기본형을 가리키는 포인터로 사용되지만 interior_ptr은 관리 힙 상에서 인스턴스가 이동하여도 올바르게 추적할 수 있는 포인터로 런타임의 지배하에 있습니다(즉 인스턴스가 관리 힙 상에서 이동하여도 괜찮다).
    - pin_ptr은 관리 힙 상의 value type을 비관리 코드에서 사용하고 싶을 때 사용한다. 당연히 이 때는 관리 힙에 있는 인스턴스가 이동하면 안되므로 인스턴스의 이동을 금지한다.
    - interipor_ptr과 pin_ptr의 같은 점 : 포인터처럼 사용할 수 있다.
    - interipor_ptr과 pin_ptr 다른 점 : interipor_ptr은 관리 코드 상에서 포인터로 사용하고, pin_ptr는 비관리 코드에 포인터로 넘길 때 사용한다.
  
  
  
## 관리 코드의 array를 비관리 코드에 포인터로 전달
첫 번째로 간단하게 array로 만든 배열을 비관리 코드의 포인터로 전달.  

```
#include <Iostream>

using namespace System;

void DumpNativeArray( int* pArrNums, int length )
{
	for( int i=0; i<length; i++ )
	{
		std::cout << pArrNums[i] << std::endl;
	}
}

int main()
{
	array< int >^ ArrNums = gcnew array<int>(3);
	ArrNums[0] = 1;
	ArrNums[1] = 2;
	ArrNums[2] = 3;

	 pin_ptr<int> pNative = &ArrNums[0];
	 DumpNativeArray(pNative,ArrNums->Length);
	 pNative = nullptr;

getchar();
	 return 0;
}
```
  
위 코드의 핵심은  
```
pin_ptr<int> pNative = &ArrNums[0];
```  
이다.  
  
pin_ptr을 사용하여 관리 힙에 할당된 객체가 이동하지 못하도록 고정한다. 고정하는 이유는 비관리 코드로 메모리 주소를 넘기기 때문에 관리 힙에서 이동이 되면 안되기 때문.
또 여기서 자세히 봐야 되는 것이 있다.  
```
pin_ptr<int> pNative = &ArrNums[0];
```
  
pNative에 &ArrNums이 아닌 &ArrNums[0]을 대입하였다. 이유는 관리 코드에서는 &ArrNums는 ArrNums의 요소가 아닌 ArrNums 오브젝트 자체를 가리키는 것이기 때문이다.
&ArrNums[0]을 대입해야 ArrNums에 들어가 있는 요소의 첫 번째 주소를 비관리 코드에 주소를 넘길 수 있다.  
  
  
  
## 관리코드의 문자열과 비관리코드의 문자열 변환
- C/C++ 문자열을 String^으로 변환
  
```
#include <msclr\marshal.h>

using namespace System;
using namespace msclr::interop;

int main()
{
		   const char* message = "Forever Visual C++";
		   String^ result1 = marshal_as<String^>( message );
		   Console::WriteLine("char -> System::String : {0}", result1);

		   const wchar_t* Wmessage = L"Visual C++이여 영원하라";
		   String^ result2 = marshal_as<String^>( Wmessage );
		   Console::WriteLine("wchar -> System::String : {0}", result2);

		   getchar();
		   return 0;
}
```
  
관리코드와 비관리코드 간의 문자열 변환에는 ‘msrshal_as’를 사용하여 마샬링한다.  
C/C++ 문자열을 마샬링하기 위해서는   
```
#include <msclr\marshal.h>
```
   
파일을 포함하고, using namespace msclr::interop; 네임스패이스를 선언한다.  
사용 방법은 아주 간단.  
```
marshal_as< 변환할 문자 타입 >( 원본 문자열 ); 
```
  
- ANSI 문자열을 관리코드의 문자열로 변환할 때는 아래와 같이 한다.
  
```
const char* message = "Forever Visual C++";
String^ result1 = marshal_as<String^>( message );
```
  
- 유니코드를 관리코드의 문자열로 변환할 때는 아래와 같이 한다.
  
```
const wchar_t* Wmessage = L"Visual C++이여 영원하라";
String^ result2 = marshal_as<String^>( Wmessage );
```
  
- STL의 string과 String^간의 변환  
이것도 marshal_as를 사용한다.  
  
```
#include <iostream>
#include <string>
#include <msclr\marshal_cppstd.h>

using namespace System;
using namespace msclr::interop;

int main()
{
		   std::string s0 = "비주얼스튜디오2010 팀블로그";
		   std::cout << "string : " << s0 << std::endl;

		   System::String^ s1 = marshal_as< System::String^ >(s0);
		   Console::WriteLine("std::sting->System::String : {0}", s1);

		   std::wstring s2 = marshal_as< std::wstring >(s1);
		   setlocale(LC_ALL, "");
		   std::wcout << "System::String->std::wstring : " << s2 << std::endl;

		   getchar();
		   return 0;
}
```
  
STL의 문자열과 변환하기 위해서는 다음의 헤더 파일을 포함해야 한다.  
  
```
#include <msclr\marshal_cppstd.h>
```
  
마샬링하는 방법은 앞에 설명한 C/C++ 문자열 변환과 같다.  
  
```
System::String^ s1 = marshal_as< System::String^ >(s0);
```
  
- 관리코드 문자열과 비관리코드 문자열간의 변환에 따른 성능  
C++로 만드는 프로그램은 보통 고성능을 원하는 프로그램이므로 보통 C++ 프로그래머는 성능에 민감하다. 마샬링은 공짜가 아니지만 많은 양을 아주 빈번하게 마샬링 하는 것이 아니면 성능에 너무 신경 쓰지 않아도 된다. 다만 기본적으로 관리코드의 문자열은 유니코드이다. 그래서 비관리코드의 문자열이 ANSI 코드라면 유니코드를 사용했을 때 보다 더 많은 시간이 걸린다(정확한 수치는 잘 모르지만 ANSI가 유니코드보다 3배정도 더 걸린다고도 한다). 그래서 관리코드와 비관리코드를 같이 사용할 때는 가능한 유니코드를 사용하는 것이 훨씬 좋다.  
  
- String^을 C/C++ 문자열로 변환  
String^을 char*나 wchr_t*로 변환하는 방법에 대해서 설명한.  
  
```
#include <string>
#include <msclr\marshal_cppstd.h>

using namespace System;
using namespace msclr::interop;

int main()
{
		   System::String^ s0 = L"비주얼스튜디오2010 팀블로그";

		   // 방법 1
		   std::string tmp = marshal_as<std::string>(s0);
		   const char* s1 = tmp.c_str();
		   std::cout << "String^ -> string : " << s1 << std::endl;

		   // 방법 2
		   const char* s2;
		   const wchar_t* s3;
		   {
					 marshal_context ctx;
					 s2 = ctx.marshal_as<const char*>(s0);
					 s3 = ctx.marshal_as<const wchar_t*>(s0);

					 std::cout << "String^ -> char* : " << s2 << std::endl;

					 setlocale(LC_ALL, "");
					 std::wcout << "String^ -> wchar_t : " << s3 << std::endl;
		   }

		   getchar();
		   return 0;
}
```
  
String^을 char*나 wchr_t*로 변환하는 방법은 두 가지가 있다.  
  
- 첫 번째 std::string 사용  
가장 간단한 방법이지만 불필요한 std::string을 사용해야 단점이 있다.  
  
```
std::string tmp = marshal_as<std::string>(s0);
const char* s1 = tmp.c_str();
std::cout << "String^ -> string : " << s1 << std::endl;
```
  
- 두 번째 marshal_context 사용  
첫 번째 방법에서 std::string을 사용한 이유는 다름이 아니고 메모리 확보 때문이다.  
마샬링을 통해서 char*와 wchar_t*에 메모리 주소를 저장한다. 문자열 그 자체를 복사하는 것이 아니다. 그래서 변환한 문자열을 저장할 메모리 주소를 확보하고 사용 후에는 해제를 해야한다. 메모리 확보와 해제를 위해서 marshal_context를 사용한다.  
marshal_context는 변환에 필요한 메모리를 확보하고, 스코프를 벗어날 때 메모리를 해제한다.  

```
const char* s2;
const wchar_t* s3;
{
		   marshal_context ctx;
		   s2 = ctx.marshal_as<const char*>(s0);
		   s3 = ctx.marshal_as<const wchar_t*>(s0);
}
```
  
String^을 C/C++ 문자열로 변환할 때는 std::string + marshal_as 나 marshal_context 둘 중 하나를 선택하여 사용한다.  
  
  
  
## parameter array
비관리코드에서 로그 기능을 구현할 때 주로 가변 길이 인수를 사용한다.  
  
```
void LOG( char* szText, ... )
{
}
```
  
위와 같은 가변 길이 인수를 C++/CLI에서는 parameter array라는 것으로 구현한다.  
  
```
void LOG( … array<Object^>^ Values )
{
	for each( Object^ value in Values )
	{
	   ….
	}
}

int main()
{
	LOG( 23 );
	LOG( 2, 1, “error” );

	return 0;
}
```
  
parameter array를 사용하면 이전 보다 안전하고, 하나의 형이 아닌 다양한 형을 인자로 받아 들일 수 있어서 유연성이 높다.  
그러나 하나의 함수에서 parameter array는 하나 밖에 사용하지 못한다. 하지만 parameter array가 아닌 형이라면 여러 개 사용할 수 있다. 다만 이 때는 parameter array는 가장 마지막 인수가 되어야 한다.  
  
```
void LOG( int nLogLevel, … array<Object^>^ Values )
{
}
```
  
  
## static 생성자, initonly, literal
- static 생성자  
static 생성자는 클래스의 생성자에서 static 멤버를 초기화 하고 싶을 때 사용한다.  
ref class, value class, interface에서 사용할 수 있다.  
  
```
#include "stdafx.h"
#include <iostream>

using namespace System;

ref class A {
public:
	static int a_;
	static A()
	{
		a_ += 10;
	}
};

ref class B {
public:
	static int b_;
	static B()
	{
//        a_ += 10; // error
		b_ += 10;
	}
};

ref class C {
public:
	static int c_ = 100;
	static C()
	{
		c_ = 10;
	}
};

int main()
{
	Console::WriteLine(A::a_);
	A::A();
	Console::WriteLine(A::a_);

	Console::WriteLine(B::b_);

	Console::WriteLine(C::c_);

	 getchar();
	return 0;
}
```
  
static 생성자는 런타임에서 호출하기 때문에 클래스 A의 멤버 a_는 이미 10으로 설정되어 있다. 그리고 이미 런타임에서 호출하였기 때문에 명시적으로 A::A()를 호출해도 실제로는 호출되지 않는다.  
클래스 B의 경우 static 생성자에서 비 static 멤버를 호출하면 에러가 발생한다.  
클래스 C의 경우 static 멤버 c_를 선언과 동시에 초기화 했지만 런타임에서 static 생성자를 호출하여 값이 10으로 설정되었다.  
  
- initonly  
initonly로 선언된 멤버는 생성자에서만 값을 설정할 수 있다. 그리고 initonly static로 선언된 멤버는 static 생성자에서만 값을 설정할 수 있다.  
  
```
ref class C
{
public:
	initonly static int x;
	initonly static int y;
	initonly int z;
	static C()
	{
		x = 1;
		y = 2;
		// z = 3; // Error
	}
	C()
	{
		// A = 2; // Error
		z = 3;
	}
	void sfunc()
	{
		// x = 5; // Error
		// z = 5; // Error
	}
};

int main()
{
	System::Console::WriteLine(C::x);
	System::Console::WriteLine(C::y);
	C c;
	System::Console::WriteLine(c.z);
	return 0;
}
```
  
- literal  
literal로 선언된 멤버는 선언과 동시에 값을 설정하고 이후 쓰기는 불가능합니다. 오직 읽기만 가능하다.   
  
```
using namespace System;
ref class C
{
public:
	literal String^ S = "Hello";
	literal int I = 100;
};

int main()
{
	Console::WriteLine(C::S);
	Console::WriteLine(C::I);
	return 0;
}
```
  
  
## 인터페이스 ( interface )
인터페이스는 비관리코드에서는 순수가상함수만을 가진 클래스와 같다. ‘interface class’라는 키워드를 사용하여 정의하면 이 클래스에는 아래와 같은 형만 멤버로 가질 수 있다.   
- 함수
- 프로퍼티
- 이벤트
  
또한 선언만 가능하지 정의는 할 수 없다.  
C++/CLI의 ref class는 C#이나 Java와 같이 다중 상속은 할 수 없지만 인터페이스를 사용하면 다중 상속(즉 인터페이스를 상속)을 할 수 있다.  
  
```
interface class IA {
public:
	void funcIA();
};
interface class IB {
public:
	void funcIB();
};
interface class IC {
public:
	void funcIC();
};
ref class A {
	int i;
};
ref class B : A, IA, IB, IC {
public:
	virtual void funcIA() {   }
	virtual void funcIB() {   }
	virtual void funcIC() {   }
};
int main() {
	B^ b = gcnew B;

	IA^ ia = b;
	IB^ ib = b;
	IC^ ic = b;
	b->funcIA();
	ia->funcIA();
	ib->funcIB();
	ic->funcIC();
	return 0;
}
```
  
  
## array 클래스에 non-CLI 오브젝트 사용
C++/CLI은 네이티브 C++과 다르게 자료구조 배열을 사용하기 위해서는 array 컨테이너를 사용한다.  
array 컨테이너는 기본적으로 non-CLI 오브젝트는 사용할 수가 없다. 그러나 non-CLI 오브젝트가 포인터라면 사용할 수 있다.  
아래는 array 컨테이너에 non-CLI 오브젝트를 사용한 경우이다.  
  
```
using namespace System;

class CNative
{
public:
	CNative()
	{
		Console::WriteLine(__FUNCTION__);
	}

	~CNative()
	{
		Console::WriteLine(__FUNCTION__);

	}
};

int main(array<System::String ^> ^args)
{
	array<CNative>^ arr = gcnew array<CNative>(2);
	return 0;
}
```
  
빌드하면 위에 이야기 했듯이 빌드 에러가 발생한다.  
그럼 이번에는 non-CLI 오브젝트의 포인터를 사용해 보겠다.  
  
```
#include "stdafx.h"
#include <iostream>

using namespace System;

class CNative
{
public:
	CNative()
	{
		Console::WriteLine(__FUNCTION__);
	}

	~CNative()
	{
		Console::WriteLine(__FUNCTION__);
	}
};

int main(array<System::String ^> ^args)
{
	array<CNative*>^ arr = gcnew array<CNative*>(2);

	for(int i=0; i<arr->Length; i++)
	{
		arr[i] = new CNative();
	}

	getchar();
	return 0;
}
```
  
위 코드를 실행하면 CNative 오브젝트의 생성자는 호출하지만 파괴자는 호출되지 않는다. 이것은 array 컨테이너는 CLI 객체이므로 GC에서 관리하지만 non-CLI 오브젝트를 포인터 타입으로 사용한 것은 GC에서 관리하지 않으므로 만약 array가 GC에서 소멸 되기 전에 array에 담겨있는 non-CLI 오브젝트를 메모리(new로 할당한)를 해제하지 않으면 메모리 해제가 되지 않아서 메모리 릭이 발생한다.  
그래서 아래와 같이 array가 GC에서 소멸되기 전에 메모리를 해제하도록 해야한다.  
  
```
#include "stdafx.h"
#include <iostream>

using namespace System;

class CNative
{
public:
	CNative()
	{
		Console::WriteLine(__FUNCTION__);
	}

	~CNative()
	{
		Console::WriteLine(__FUNCTION__);
	}
};

int main(array<System::String ^> ^args)
{
	array<CNative*>^ arr = gcnew array<CNative*>(2);

	for(int i=0; i<arr->Length; i++)
	{
		arr[i] = new CNative();
	}

	for(int i=0; i<arr->Length; i++)
	{
		delete arr[i];
	}

	getchar();
	return 0;
}
```
  
  
## 델리게이트에 비관리 함수를 할당하기
  
```
#pragma unmanaged

extern "C" void printf(const char*, ...);

class A {

public:

   static void func(char* s) {

	  printf(s);

   }

};



#pragma managed

public delegate void func(char*);



ref class B {

   A* ap;



public:

   B(A* ap):ap(ap) {}

   void func(char* s) {

	  ap->func(s);

   }

};



int main() {

   A* a = new A;

   B^ b = gcnew B(a);

   func^ f = gcnew func(b, &B::func);

   f("hello");

   delete a;

}
```
  
위 코드 중 #pragma unmanaged 지시어 이하는 컴파일러에서 비관리코드로 취급한다. 그리고 #pragma managed 이하는 관리코드로 취급한다  
  
  
  
## 순수 가상 함수
C++/CLI에서는 ‘abstract’라는 키워드를 사용한다.  
  
```
public ref class Server
{
   .....
   virtual void OnAccept() abstract;
};
```
  
그런데 이렇게만 하면 클래스는 abstract가 아니라는 경고가 나온다.  
경고를 없애고 싶다면 클래스에도 abstract를 붙여준다.  
  
```  
public ref class Server abstract
{
   .....
   virtual void OnAccept() abstract;
};
```
  
  
## char* -> 관리코드, 관리코드 -> char*
char*는 문자열을 뜻하는 것이 아니고 char형의 포인터를 뜻한다.  
C++의 네트웍 프로그래밍에서는 네트웍으로 데이터를 주고 받을 때 char* 타입으로 주고 받는다.  
  
```
// 보내기
char* pPacket;
…
Send(pPacket, nLength);

// 받기 
char* pBuffer = new char[MAX_BUFFER];
int nLength = Receive( pBuffer );
```
  
C++/CLI에서 C++로 만든 네트웍 라이브러리를 사용하는 경우 필연적으로 char*을 관리코드로 바꾸고, 관리코드를 char*로 바꾸어야 하는 경우가 있을 것이다.  
  
- char* -> 관리코드  
C++/CLI에서 char*는 array< Byte >로 마샬링 하면 된다.   
  
```
// 비관리코드
int nPacketSize = 34;
char* pPacket = new char[34];

// 관리코드
array< Byte >^ byteArray = gcnew array< Byte >(nPacketSize);
System::Runtime::InteropServices::Marshal::Copy( (IntPtr)pPacket, byteArray, 
0, nPacketSize );
```
  
- 관리코드 -> char*
  
```
// 관리코드
array<Byte>^ SendData;
….

// 비관리코드
int nLength = SendData->Length;
pin_ptr<Byte> pData = &SendData[0];

char* pBuffer = new char[ nLength ];
CopyMemory( pBuffer, pData, nLength );
```
  
참고로 C++/CLI에서는 바이트형 배열 array<Byte>가 C#에서는 byte[] 이 된다.  


