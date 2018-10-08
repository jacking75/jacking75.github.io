---
layout: post
title: C++- Poco 라이브러리 Visual Studio 2017 에서 빌드 하기
published: true
categories: [C++]
tags: c++ poco vc
---
poco 라이브러리를 Visual Studio 2017 에서 빌드하려면 아직 쉽게 되지 않는다.   
아래처럼 Visual Studio 2017 설치 메뉴에 있는 콘솔 프로그램을 실행해서 빌드 파일을 실행해야 한다.  
  
![](/images/cpp/poco_2017_build.PNG)    
  
<pre>
buildwin.cmd 150 rebuild static_md both x64
buildwin.cmd 150 rebuild static_md both Win32
</pre>
  
<pre>
32bit: buildwin 110 build all both Win32 nosamples notests devenv
64bit: buildwin 110 build all both x64 nosamples notests devenv
</pre>  
  
<br>  
  
poco 라이브러리 1.8 빌드 시 만약 VC의 옵션중 std:c++latest 혹은 std::C++17 이 있으면 'std::pointer_to_unary_function'을 찾지 못한다는 에러가 나온다.  
이유는 std::pointer_to_unary_function은 C++17 에서 제거 되었기 때문입니다.  
  
<br>  
    
poco 빌드와 관련해서 아래 글도 참고 하기 바란다.  	
[Poco Library 빌드하기 2](https://neodreamer-dev.blogspot.kr/2017/11/poco-library.html)  
  