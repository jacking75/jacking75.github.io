---
layout: post
title: 클래스 멤버 함수 포인터 배열 사용하기
published: true
categories: [C++]
tags: c++ function_pointer
---
함수 포인터 관련 검색을 하면 C 버전이 많이 나오고, 배열 타입을 사용하는 경우는 별로 없어서 공유한다.  
(사실 최신 C++에서는 std::function을 사용하면 되기 때문에 이걸 사용하지 않아도 된다)

  
**선언**  

```
bool(TEST::*m_pFunc[4])(const int, bool);
```

    
**멤버 함수 연결**  

```
TEST()
{
	m_pFunc[0] = &TEST::func1;
	m_pFunc[1] = &TEST::func2;
}
```
  
   
**함수 포인터 배열 호출**  

```
bool IsTest(int i) { return (this->*m_pFunc[i])(1, false); }
```
  
  
아래는 예제 코드이다.

```
#include <iostream>

class TEST
{
public:
	TEST()
	{
		m_pFunc[0] = &TEST::func1;
		m_pFunc[1] = &TEST::func2;
	}

	bool(TEST::*m_pFunc[4])(const int, bool);

	bool IsTest(int i) { return (this->*m_pFunc[i])(1, false); }

private:
	bool func1(const int a, bool isFlag) { return true; }
	bool func2(const int a, bool isFlag) { return false; }
};


int main()
{
	TEST test;

	std::cout << test.IsTest(0) << std::endl;
	std::cout << test.IsTest(1) << std::endl;

	return 0;
}
```
