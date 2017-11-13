---
layout: post
title: Visual Studio - VC++ 의 디버그 모드 실행이 느린 경우 빠르게 하기
published: true
categories: [Visual Studio]
tags: vs vc c++ vs2017
---
VC++에서 디버그 모드로 실행할 경우 프로그램 실행 속도가 엄청나게 느려질 수 있다. (특히 PC 온라인 게임 클라이언트)  
  
이런 경우 컴파일러 옵션을 조정해서 실행 속도를 올릴 수 있다.  
디버그 모드에서는 최적화 옵션이 off로 되어 있는데 이것을 on으로 한다.  
그런데 최적화 옵션을 on으로 하면 아마 디버깅에 문제가 발생할 것이다.  
그래서 솔루션 전체의 최적화 옵션을 on으로 하지 않고, 디버깅에 사용할지 않을 소스 파일들만 최적화 옵션을 on 으로 한다.  
  
솔루션 탐색기에서 디버깅에 사용하지 않을(즉 브레이크 포인터에 들어가지 않을) 소스 파일을 선택 후 마우스 오른쪽 클릭으로 속성 창을 연다. 
  
<pre>
[C/C++] -> [최적화] - [최적화] 부분을 Ox 또는 O2로 한다.
[C/C++] -> [코드 생성] - [기본 런타임 검사] 부분을 기본값 으로 한다.
</pre>  


Visual Studio C++ tip:  Quickly want to debug some code without a rebuilt #pragma optimize( "", off )
..code..
#pragma optimize( "", on )
https://twitter.com/MittringMartin/status/912817354827907073