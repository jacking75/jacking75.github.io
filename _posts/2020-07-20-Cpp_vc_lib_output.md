---
layout: post
title: C++ - 라이브러리 출력 이름 변경하기
published: true
categories: [C++]
tags: c++ vc vc++ lib
---
[출처](https://erio-nk.hatenadiary.org/entry/20110519/1305819066  )  
  
라이브러리 출력 이름을  
```
MyLibrary_x86_Debug.lib
```  
라고 하고 싶다.  
또 아래처럼 디렉토리 구성으로 하고 싶다.   
<pre>
MyLibrary
    MyLibrary.sln
    MyLibrary
        MyLibrary.vcxproj
lib
    MyLibrary_x86_Debug.lib
    MyLibrary_x64_Debug.lib
    MyLibrary_x86_Release.lib
    MyLibrary_x64_Release.lib
</pre>  
  
으 프로젝트는 라이브러리를 만드는 것이다.    
  
  
## 구성 프로퍼티 -> 일반
출력 디렉토리를 바꾼다.  
<pre>
$(SolitionDir)..\lib\
</pre>  
  
  
타겟 이름을 바꾼다.  
<pre>
$(SolutionName)_$(Platform)_$(Configuration)
</pre>  
  
출력 디렉토리나 중간 디렉토리의 끝에 `\`을 넣지 않으면 경고가 나올 수 있다.  
