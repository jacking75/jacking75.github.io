---
layout: post
title: Boost 라이브러리 설치하기
published: true
categories: [C++]
tags: c++ boost
---
# 설치
## Linux
1. Boost 라이브러리 공식 사이트에서 다운로드 후 압축 풀기
2. 압축을 푼 디렉토리로 이동 후 아래처럼 입력
3. `$ ./bootstrap.sh`
4. 생성된 b2를 사용하여 Boost 라이브러리 빌드하기
5. `$ ./b2 toolset=gcc link=static threading=multi address-model=64`
6. 빌드가 끝나면 Boost 라이브러리 디렉토리 안의 stage/lib 디렉토리에 빌드 릴리즈용 lib 파일이 만들어져 있다
  
  
아래는 디버그 버전으로 빌드하고 생성될 디렉토리를 지정한다.  
```
./b2 toolset=gcc link=static threading=multi address-model=64 variant=debug stagedir=./stage\lib64Debug
```  
릴리즈 버전을 명시적으로 지정하고 싶다면  
```
./b2 toolset=gcc link=static threading=multi address-model=64 variant=release
```  
  
  
Clang  
```
./b2 toolset=clang link=static,shared cxxflags="-std=c+11 -stdlib=libstdc" linkflags="-stdlib=libstdc+" threading=multi
```
  
  
  
## Windows
VC용으로 이미 빌드된 Boost 라이브러리 Lib 파일을 얻을 수 있다. boost 라이브러리 버전, VC 버전 별로 다 있다.    
[다운로드](http://sourceforge.net/projects/boost/files/boost-binaries/ )  
  
  
### b2의 옵션 설명  
[출처](https://boostjp.github.io/howtobuild.html )  
  
#### toolset
하나의 머신에 서로 다른 종류의 컴파일러가 인스톨 되어 있는 경우는 toolset 명령어로 지정할 수 있다. 예를 들면:  
borland : Borland사의 컴파일러  
dmc : Digital Mars사의 컴파일러  
darwin : Apple에서 손을 댄 gcc 컴파일러(Mac OS)  
gcc : GNU 프로젝트에 의한 컴파일러  
intel : Intel사 컴파이럴  
msvc : Microsoft사 컴파일러  
로 지정할 수 있다.  
  
msvc-9.0 (Visual C++ 2008), msvc-10.0 (Visual C++ 2010)와 같이 버전 지정도 할 수 있다.  
   
   
#### link
이것은 static, shared 라이브러리를 만들지를 지정하는 명령어  
link=static,shared 와 같이 사용한다.  
lib, dll (Windows)  
a, dylib (Mac OSX)  
a, so (Other Systems)  
와 같은 라이브러리 파일을 생성한다.  
  
  
#### threading 
multi: 멀티 스레드 라이브러리 생성  
single: 싱글 스레드 라이브러리 생성  
  
  
#### variant
debug: 디버그 빌드를 생성  
release: 릴리즈 빌드를 생성  
  
  
참고 글  
[(한글)Install boost library for Linux](http://gilgil.net/?document_srl=7273 )  
  
  
  
# 간단 테스트
## linux, g++
아래 코드가 있는 디렉토리 상위 디렉토리에 thirdparty/boost 디렉토리에 빌드된 boost가 있다고 가정한다.  
hello.cpp  
```
#include <boost/regex.hpp>
#include <iostream>
#include <string>
 
int main()
{
    std::string line;
    boost::regex pat("^Subject: (Re: |Aw: )*(.*)");
 
    while(std::cin)
    
        std::getline(std::cin, line);
        boost::smatch matches;
 
        if(boost::regex_match(line, matches, pat))
            std::cout << matches[2] << std::endl;
    
 
    return 0;
}
```  
  
Makefile  
```
CC=g++
CPPFLAGS=-I../thirdparty/boost
LDFLAGS=-L../thirdparty/boost/stage/lib
LDLIBS=-lboost_regex
 
all:hello
 
clean:
        rm -rf hello
        rm -rf *.o
```  
  
실행이 잘 되었다면 boost 빌드가 잘 된 것이다.  
  
  
## linux, clion
위와 같은 코드를 빌드한다고 가정한다.  
기본 프로젝트를 만들면 cmake 파일의 내용은 아래와 같다.  
  
```  
cmake_minimum_required(VERSION 3.9)
project(hello_boost_clion)
 
set(CMAKE_CXX_STANDARD 17)
 
add_executable(hello_boost_clion main.cpp)
```
  
아래와 같이 변경한다  
```
cmake_minimum_required(VERSION 3.9)
project(hello_boost_clion)
 
set(CMAKE_CXX_STANDARD 17)
 
set(BOOST_ROOT $ENV{HOME}/dev/cpp/thirdparty/boost)
set(Boost_INCLUDE_DIR ${BOOST_ROOT})
set(Boost_LIBRARY_DIR ${BOOST_ROOT}/stage/lib)
 
find_package(Boost COMPONENTS regex REQUIRED)
 
include_directories(${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIR})
 
add_executable(hello_boost_clion main.cpp)
target_link_libraries( hello_boost_clion ${Boost_LIBRARIES} )
```  
  
  
# Boost 라이브러리를 어떤 컴파일러 어떤 버전으로 빌드했는지 알고 싶을 때
BOOST_COMPILER 라는 지시어를 사용하면 컴파일러 종류와 버전을 알려준다.  
```
#include <iostream>
#include <boost/config.hpp>
 
int main()
{
    std::cout << BOOST_COMPILER << std::endl;
    return 0;
}
```
  
  
  
# 참고
[Boost 빌드 및 각종 IDE에서 사용하기](https://github.com/jacking75/CookBookBoostCpp/blob/master/build.md )  
  
