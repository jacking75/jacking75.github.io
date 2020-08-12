---
layout: post
title: C++ - VC++ 에서 링크 에러가 발생하는 경우
published: true
categories: [C++]
tags: c++ vc link
---
VC++을 사용할 때 링크 오류가 발생하여 시간을 허비하는 경우가 있다.  
보통 아래의 이유로 링크 오류가 발생하는 경우가 많다.  
 
## 문자 코드의 취급이 같지 않다.
Use Unicode Character Set, Use Multi-Byte Character Set  
   
   
## [C/C++][Code generation][Runtime Library]의 취급이 같지 않다.
- Release 모드| Debug 모드
- 정적 링크를 사용할 경우 Multi-threaded(/MT) | Multi-threaded Debug(/MTd)
- DLL을 사용할 경우 Multi-threaded DLL(/MD) | Multi-threaded Debug DLL(/MDd)
   
   
## 링크 라이브러리의 디렉토리를 지정하지 않았다.
[VC++ Directories][library Directories]  
   
   
## 링크하는 설정 lib 파일이 아직 준비되지 않았다.
- Debug 모드 or Release 모드
- 문자 코드
- MultiThread 취급
- 32bit Or 64bit
 
## 복사한 설정에서 잘못된 의존성·설정이 된 경우
  
## 설정에 다른 버전의 컴파일러가 사용 되었을 때  
  