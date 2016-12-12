---
layout: post
title: 클래스 멤버 함수 포인터 배열 사용하기
published: false
date: 2017-12-12
categories: [C++]
tags: c++
---

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
