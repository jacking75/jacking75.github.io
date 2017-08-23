---
layout: post
title: GitHub의 markdown 문법의 코드 블럭에서 문법 하이라트 가능한 언어 리스트
published: true
categories: [etc]
tags: github markdown
---
Github의 markdown 에서 코드를 기술할 때  
<pre>
```
```  
</pre>  
라는 블럭으로 코드를 감싼다.  
  
여기에 더해서 코드 블럭에 있는 코드의 문법 하이라이트 기능을 사용하고 싶다면 해당 언어 키워드를 적으면 된다.  
swift의 경우는 아래와 같다.  
<pre>
```Swift
```
<pre>    
  
[Github에서코드 블럭 문법 하이라이트 지원 언어 리스트 ](https://github.com/github/linguist/blob/master/lib/linguist/languages.yml)  
  
참고로 C++은 C++ 혹은 cpp, C#은 C# 혹은 csharp 이다.  
  
문법 하이라이트를 사용하지 않은 경우    
```
#include <iostream>

int main()
{
	std::cout << "jacking75" << std::endl;
	return 0;
}
``` 
  
문법 하이라이트를 사용한 경우    
<pre>
```C++
#include <iostream>

int main()
{
	std::cout << "jacking75" << std::endl;
	return 0;
}
```
<pre>  

<br>  
  
내 블로그의 markdown 문법은 github의 markdown과 달라서 문법 하이라트를 사용할 수 없다. ㅠㅠ