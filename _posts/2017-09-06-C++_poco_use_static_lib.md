---
layout: post
title: C++- Poco 라이브러리를 VC++에서 static 라이브러리로 사용할 때
published: true
categories: [C++]
tags: c++ poco vc
---
Poco 라이브러리를 VC++ 에서 static 라이브러리로 사용할 때 lib 파일을 찾을 수 없다는 링크 에러를 만나는 경우는 아래의 선언을 추가하면 된다.  
```C++
#define POCO_STATIC 
```  
  
<br>  
    
Poco 라이브러리는 default로는 동적라이브러리를 사용하는 것으로 설정 되어 있다. 
그래서 위의 선언으로 정적 라이브러리 사용을 알려준다.  
  
> The default is to link the POCO C++ Libraries dynamically (DLL).
> To link statically, the code using the POCO C++ Libraries must be compiled with the preprocessor symbol POCO_STATIC defined.  
  
[참고](https://pocoproject.org/docs/99150-WindowsPlatformNotes.html)  
  