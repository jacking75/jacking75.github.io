---
layout: post
title: LLVM 이란?
published: true
categories: [번역]
tags: llvm clang
---
[원문](https://dev.classmethod.jp/server-side/about_llvm/)  
  
LLVM 프로젝트는 모듈러와 재이용 가능한 컴파일러와 툴 체인 기술의 집합이다.  
이 이름에도 불구하고 LLVM은 기존 VM과는 거의 관계가 없다.  
LLVM이라는 이름은 머리 글자를 딴 것이 아니라 프로젝트 이름이다.  
  
LLVM은 모던하고 SSA 베이스한 임의의 프로그래밍 언어의 정적 컴파일과 동적 컴파일을 모두 지원할 수 있는 컴파일 전략을 제공하는 것을 목적으로 한 일리노이 대학의 연구 프로젝트에서 시작되었다. 그 뒤 LLVM은 몇 개의 서브 프로젝트로 구성되는 umbrella project로 성장했다.  
대부분은 학술 연구는 물론 폭넓은 분야의 상업 용이나 오픈 소스 프로젝트에 의한 프로덕션에서 사용되고 있다.  
LLVM 프로젝트의 소스 코드는 "UIUC" BSD-Style 라이선스이다.  
  
LLVM의 주요 서브 프로젝트  
1. LLVM Core 라이브러리  
  source 나 target 이라는 독립한 현대적인 옵티마이저와 많은 일반적인 CPU(일반적이 아닌 것도 있다)의 코드 생성을 지원한다.  
  이들 라이브러리는 LLVM intermediate representation(LLVM IR)라고 알려진 잘 정의된 코드 표현을 중심으로 구성된다.  
  LLVM Core 라이브러리는 잘 문서화되어 있으므로 당신 자신이 언어를 작성(또는 기존의 컴파일러를 가져와서), 옵티마이저, 코드 생성기로 LLVM을 사용하는 것은 특히 간단한다.  
2. Clang  
  LLVM 네이티브인 C/C++/Objective-C 컴파일러로 놀라울 만큼 빠른 컴파일을 제공하는 것을 목적으로 하고 있다.  
3. LLDB project  
  LLVM과 Clang에 의해서 제공된 라이브러리 상에서 네이티브한 디버거를 제공한다.
4. ibc++ and libc++ ABI projects  
  C++ 표준 라이브러리의 표준 준수와 고성능한 구현을 제공한다.  
5. compiler-rt project
6. OpenMP subproject
7. polly project
8. libclc project
9. klee project
10. SAFECode project
11. lld project  
  
  
## Features
### LLVM Features
C와 C++ 컴파일러의 이야기.  
  
### Strengths of the LLVM System
1. LLVM은 심플한 저 수준 언어를 사용한다.
2. LLVM에는 C와 C++용 프론트엔드가 포함된다. Java, Scheme, 기타 언어 프론트엔드는 개발 중이다.
3. LLVM에는 굉장히 옵티마이저가 포함된다.
4. LLVM은 life-long compilation model을 지원한다.
5. LLVM은 accurate garbage collection을 완전 지원한다.
6. LLVM code generator 라는 것이 있다.
7. 이후에는 기능적인 이야기가 없어서 생략.
    
프론트엔드가 각 프로그램 언어를 해석하는 부분, 중간에 있는 옵티마이저가 "LLVM Core 라이브러리", 백엔드가 각 프로세서용 기계 코드를 출력하는 부분이다.  
"LLVM Core 라이브러리"의 설명에도 있었던 source와 target은 아마 이 프론트엔드와 백엔드로 말하는 것이다.  
  
  
### LLVM Audience
이 장은 LLVM이 누구 전용인지 쓰고 있다.  
  
- 컴파일러 연구자
- VM 연구자
- 아키텍처 연구자
- 보안 연구자
- 컴파일러의 프로토 타입에 관심이 있는 강사나 개발자
- 퍼포먼스를 추구하고 싶은 최종 사용자
  
  
### 질문답
Q1. LLVM은 컴파일러 기반라고 하는데 컴파일러 기반은 뭐죠?  
A1. 임의의 프로그래밍 언어에서 임의의 프로세서의 기계 코드로 컴파일할 수 있는 기반이라고 생각한다.  
LLVM을 이용함으로써 중간 언어의 최적화 처리와 다양한 아키텍쳐에 대응한 기계어 코드 생성 처리를 재활용 할 수 있는 것이 장점이 된다고 생각한다.  
  
Q2. LLVM은 컴파일러 기반이며 VM이 없다고 하지만 LLVM에 포함되는 JIT 컴파일러는 VM과 다른가?   
A2. (이것은 LLVM의 설명에 쓰인 건 아니지만)VM은 프로세서를 에뮬레이트 하는 것에 대해서, JIT 컴파일러는 기계 언어로 컴파일하고 실행한다는 점이 다르지 않을까 생각한다.    
예를 들어 JVM(Java 가상 머신)는 JIT 컴파일러를 이용하여 실행하고 있으면서 VM으로 불리니 LLVM도 JIT컴파일러 이면서 MV가 아니라는 표현이 꼭 올바르지는 않은듯.   
    
