---
layout: post
title: C++ - SSO(Small-string optimization)
published: true
categories: [C++]
tags: c++ string sso
---
보통 std::string는 문자열을 확보할 때 동적으로 메모리를 확보하지만 작은 크기의 문자열의 경우는 동적 할당은 성능적으로 낭비이므로 동적 할당을 하지 않는 최적화가 구현 되어 있고 이것을 **SSO** 라고 한다. 
  
아래처럼 std::string 안에 문자열을 저장할 수 있는 버퍼를 가진다.  
```
class string{
char sso[16];
...
};
```  
  
SSO의 크기는 각 컴파일러 마다 다르다.  
VC의 경우 아래 코드를 실행하면 디버그 모드에서는 40, 릴리즈 모드에서는 32가 출력된다.  
```
#include<string>
#include<iostream>

int main()
{
	std::cout << sizeof(std::string) << std::endl;
	return 0;
}
```
      
<br>  
<br>  
  	  
[참고](https://qiita.com/melpon/items/caf4a2bd74968db7032f)	  