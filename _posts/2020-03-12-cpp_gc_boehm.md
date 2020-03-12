---
layout: post
title: C++ - GC(Boehm) 라이브러리 사용하기
published: true
categories: [C++]
tags: c++ gc boehm
---
[출처](https://qiita.com/Gajumaru/items/195adf964d3932e6dc57 )   
  
C++에서 GC(가베지컬렉션)를 사용할 수 있게 해주는 라이브러리 이다.  
  
## 1. 소스 코드 다운로드
[다운로드 페이지](https://github.com/ivmai/bdwgc/wiki/Download )에서 다운로드.  
특별한 이유가 없으면 'recent stable release" 버전을 다운로드.  
압축을 푼다.  
  
  
## 2. 솔루션 파일 만들기
솔루션을 만들기 위해 CMake를 사용한다.  
  
설치한 후 명령 프롬프트를 열고 명령을 친다.  
`cd C:\gc-8.0.4.tar\gc-8.0.4` (프로젝트 위치로 이동)  
`cmake -D enable_cplusplus=ON` (C++용 솔루션 파일 작성)
명령이 완료되면 gc.sln 이 만들어진다.  
  
위 방법 이외에 CMake-GUI 툴로 솔루션을 만들 수 있다.  
  
  
## 3. 빌드해서 .dll과 .lib를 만든다
기본 값을 빌드하면 모든 프로젝트가 컴파일된다.  
생성된 gcmt-dll.dll 과 gcmt-dll.lib 을 사용한다.  
  
  
## 4. 사용하고자 하는 프로젝트에 추가
사용하고 싶은 프로젝트를 만들고 프로젝트 (P) -> 속성(E) 을 열고 방금 라이브러리를 프로젝트에 추가한다.  
- 구성 속성 -> C/C++ 추가  에서 포함 디렉터리에 include 폴더의 경로를 추가.
    - 예: `C:\gc-8.0.4.tar\gc-8.0.4\include`
- 구성 속성 -> 링커   추가 라이브러리 디렉토리에 gcmt-dll.lib 이 들어 있는 디렉토리를 추가.
- 구성 속성 -> 링커 -> 입력   추가 종속성에 gcmt-dll.lib을 추가.
- 실행 시 gcmt-dll.dll 이 필요하기 때문에 빌드한 exe가 있는 장소 등으로 이동하여 둔다.
  
  
  
5. 시도
main.cpp  
```
#include <iostream>
#include <gc_cpp.h>

int ccnt;
int dcnt;

class Hoge : public gc_cleanup
{
public:
  Hoge() {
   ccnt++;
  }
  ~Hoge() {
    dcnt++;
  }
};

int main()
{
  for (int i = 0; i < 100000; i++) {
    Hoge *h = new Hoge();
  }
  std::cout<<"생성자가호출된 횟수: "<<ccnt<<std::endl;
  std::cout<<"소멸자 호출된 횟수: "<<dcnt<<std::endl;
  return 0;
}  
```
  