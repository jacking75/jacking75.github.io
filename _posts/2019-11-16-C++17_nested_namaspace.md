---
layout: post
title: C++17 - 중첩된 namespace 선언
published: true
categories: [C++]
tags: c++ c++17 namespace
---
C++14 까지 
```
namespace A
{
	namespace B
	{
		class AA {};
	}
}
```  
  
C++17 부터는  
```
namespace A::B
{
	class AA {};

}
```  
  
    
      