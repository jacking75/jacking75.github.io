---
layout: post
title: VS2017 에 포함되어 있는 C++ 컴파일러 버전 매크로
published: true
categories: [Visual Studio]
tags: vs vc c++ vs2017
---
VS2017 에 포함되어 있는 C++ 컴파일러의 버전 매크로는 

<pre> 
_MSC_VER 1910  
</pre>  
    
참고로 VS 2015는 1900.  
  
MSVC 런타임은 2015와 바이너리 호환. 
그리고 VS 2017를 설치할 때 옵션 선택에 의해 VS 2015.3의 MSVC도 별도 추가할 수 있다.
  


출처: https://blogs.msdn.microsoft.com/vcblog/2017/03/07/binary-compatibility-and-pain-free-upgrade-why-moving-to-visual-studio-2017-is-almost-too-easy/