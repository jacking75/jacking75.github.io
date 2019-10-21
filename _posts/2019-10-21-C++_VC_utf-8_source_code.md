---
layout: post
title: C++ - Visual C++에서 UTF-8 소스 코드를 사용하는 방법
published: true
categories: [C++]
tags: c++ g++ gcc Ubuntu Ubuntu18
---
출처: https://qiita.com/moutend/items/601ca163ecece7c4afeb  

## 전제
최신 msbuild가 필요하다. VisualStudio 2017 Community Edition을 설치하면 자동으로 설치 될 것이다.  
경로가 통하지 않는 경우 다음 명령을 실행 msbuild을 사용할 수 있게하자.  
```
set PATH=%PATH%;%PROGRAMFILES(x86)%\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin
```
  
  
## UTF-8에 대한 대응 방법
UTF-8을 지원하려면 다음 단계를 취할 필요가 있다.
1. UTF-8로 인코딩 된 소스 코드를 컴파일 할 수 있도록 설정  
2. 명령 프롬프트에 제대로 UTF-8 문자열을 표시한다  
  
  
## 1. UTF-8로 인코딩 된 소스 코드를 컴파일 할 수 있도록 설정
우선 UTF-8로 인코딩 된 소스 코드를 cl.exe로 컴파일 할 수 있도록 설정한다.
또한 소스 코드에 BOM을 붙일 필요는 없다. cl.exe에 /utf-8 또는 /source-charset:utf-8 /execution-charset:utf-8 플래그를 지정하면 UTF-8로 인식된다.  

### .vcxproj 파일
VisualStudio에서 프로젝트를 만들 때 자동으로 .vcxproj 라는 여러가지 설정을 기록하는 XML 파일이 생성된다. 물론 .vcxproj 파일은 수동으로도 만들 수 있다.  
.vcxproj 파일에 대한 자세한 내용은 설명을 생략한다. 샘플 프로젝트의 hello.vcxproj를 열고 대략적인 구조를 파악하면 충분하다.  
  
  
### AdditionalOptions 태그
AdditionalOptions 태그로 cl.exe에 전달할 매개 변수를 지정할 수 있다. hello.vcxproj를 열고 AdditionalOptions이라는 태그가 적혀 있는지 확인한다.  
```
<ClCompile>
  ...
  <AdditionalOptions>/utf-8</AdditionalOptions>
</ClCompile>
```
  
  
### 빌드
샘플 프로젝트를 빌드하려면 다음 명령을 실행한다.  
  msbuild hello.vcxproj  
그러면 Debug 폴더 안에 hello.exe이 생성된다.  
명령 프롬프트는 CP932(Shift-JIS)로 설정되어 있어도 "안녕하세요!"라고 표시 될 것이다. chcp 65001에서 코드 페이지를 UTF-8로 설정할 필요가 없다.  
  
  
## 2. 명령 프롬프트에 제대로 UTF-8 문자열을 표시한다
.vcxproj 설정만으로는 충분하지 않다. std::cout에서 문자열을 표시하면 글자가 깨진다.  
그래서 깨지는 것을 피할 수 있도록 쓰는 방법을 해야한다. 포인트는 3개이다.  
  
### main.cpp
샘플 프로젝트의 main.cpp를 연다. 내용은 다음과 같이 되어 있다.  
```
#include "stdafx.h"
#include "main.h"

int main() {
  // 포인트 1
  SetConsoleOutputCP(CP_UTF8);

  // 포인트 2
  setvbuf(stdout, nullptr, _IOFBF, 1024);

  // 포인트 3
  std::string text = u8"Hello, World!\n안녕하세요 세계!\n";
  std::cout << text;

  return 0;
}
```
  
  
### 포인트 1 SetConsoleOutputCP
이 함수에서 명령 프롬프트 코드 페이지를 지정할 수 있다. 매번 chcp 65001 이라는 명령을 박아서 명령 프롬프트 코드 페이지를 UTF-8로 전환 할 필요가 없다.  
  
### 포인트 2 setvbuf
버퍼링을 사용하지 않으면 UTF-8로 인코딩된 문자열이 끊어져 버리기 때문에, 이 함수의 호출이 필요하다. 1024라는 수치는 적당히 결정 것이므로 개발하는 프로그램에 따라 조정하자.  
  
### 포인트 3 string 타입과 u8 접두사
C++에서 문자열 앞에 타입을 나타내는 접두사를 지정해야한다. 또한 재사용 가능한 코드로 하기 위해 const wchar_t* 같은 타입은 아니고 std::string를 사용해야한다.  
  
  
  
## 정리
1. .vcxproj에서 cl.exe에 /utf-8 플래그를 지정한다.
2. SetConsoleOutputCP에서 코드 페이지를 지정한다
3. setvbuf에서 버퍼링을 사용한다
4. 문자열 앞에 u8 접두사를 붙인다
5. string 타입을 사용한다
  
  


