---
layout: post
title: Poco 라이브러리 설치하기
published: true
categories: [C++]
tags: c++ Poco
---
# Linux
- 공식 사이트에서 최신 버전을 다운로드 한다. 
- 압축을 풀고, 빌드한다.
- 아래는 64비트, static 라이브러리, 테스트와 샘플은 제외 하는 설정으로 빌드 한다
  
```  
$ tar zxvf poco-1.7.8p2-all.tar.gz
$ cd poco-1.7.8p2-all/
  
$ export OSARCH_64BITS=1
$ ./configure --static --no-tests --no-samples
$ make
```
   
설정 정보는 아래 처럼 더 추가할 수 있다. ODBC는 제외하고, mysql 라이브러리를 사용하기 위해 path를 지정.  
```
./configure --omit=Data/ODBC,Data/SQLite --prefix=/usr --cflags=-fPIC --cflags=-std=c++11 --static --shared  --no-tests --no-samples
```
   
   
빌드가 끝나면 poco 디렉토리(위에서 압축을 푼)에 라이브러리가 만들어져 있다.  
`lib/Linux/x86_64`  
디버그 버전은 라이브러리 이름 뒤에 'd'가 붙고, 64비트 버전은 뒤에 '64'가 붙는다.  
  
  
## 샘플 예제 - Makefile
sample.cpp  
```
#include <iostream>
#include <Poco/RegularExpression.h>
 
int main() {
    Poco::RegularExpression regexp("^[a-zA-Z]+");
 
    std::string buf;
    regexp.extract("ABC123", buf);
    std::cout << buf << std::endl; //=> ABC
 
    return 0;
}
```
  
Makefile  
```
CXX=g++
CXXFLAGS=-I/mnt/e/linux/dev/c++/thirdparty/poco/Foundation/include
LDFLAGS=-L/mnt/e/linux/dev/c++/thirdparty/poco/lib/Linux/x86_64
LDLIBS=-lPocoFoundation64
 
all:sample
 
clean:
    rm -rf sample
    rm -rf *.o
``` 
  
  
## 샘플 예제 - clion
위와 같은 코드를 빌드한다고 가정한다.  
기본 프로젝트를 만들면 cmake 파일의 내용은 아래와 같다.  
```
cmake_minimum_required(VERSION 3.9)
project(hello_poco_clion)
 
set(CMAKE_CXX_STANDARD 17)
 
add_executable(hello_poco_clion main.cpp)
```
   
아래와 같이 변경한다  
```
cmake_minimum_required(VERSION 3.9)
project(hello_poco_clion)
 
set(CMAKE_CXX_STANDARD 17)
 
 
set(POCO_ROOT $ENV{HOME}/dev/cpp/thirdparty/poco)
set(Poco_Foundation_INCLUDE_DIR ${POCO_ROOT}/Foundation/include)
set(Poco_LIBRARY_DIR ${POCO_ROOT}/lib/Linux/x86_64)
 
FIND_LIBRARY(Poco_LIBRARIES NAMES PocoFoundation PocoNet PATH_SUFFIXES ${Poco_LIBRARY_DIR})
 
include_directories(${Poco_Foundation_INCLUDE_DIR})
link_directories(${Poco_LIBRARY_DIR})
 
 
add_executable(hello_poco_clion main.cpp)
target_link_libraries( hello_poco_clion PocoFoundation64 )
```  
  
  
## 샘플 예제 - CMake
  
```  
$ mkdir PocoTest
$ cd PocoTest
$ touch HelloPoco.cc
```
   
HelloPoco.cc  
```
#include "Poco/Thread.h"
#include "Poco/Runnable.h"
#include <iostream>
 
class HelloRunnable : public Poco::Runnable
{
    virtual void run()
    {
        std::cout << "Hello, POCO!" << std::endl;
    }
};
 
int
main(int argc, char** argv)
{
    HelloRunnable runnable;
 
    Poco::Thread thread;
    thread.start(runnable);
    thread.join();
 
    return 0;
}
``` 
  
```
$ cd PocoTest
$ touch CMakeLists.txt
```
  
CMakeLists.txt  
```
CMAKE_MINIMUM_REQUIRED (VERSION 2.8)
LIST (APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMakeModules)

FIND_PACKAGE (POCO)
IF (POCO_FOUND)
  INCLUDE_DIRECTORIES (POCO_INCLUDE_DIRS)
  ADD_EXECUTABLE (HelloPoco HelloPoco.cc)
  TARGET_LINK_LIBRARIES (HelloPoco ${POCO_FOUNDATION})
ENDIF (POCO_FOUND)
```
   
```   
$ mkdir CMakeModules
$ cd CMakeModules
$ touch FindPOCO.cmake
```
  
FindPOCO.cmake  
```
FIND_PATH (POCO_INCLUDE_DIR Poco/Poco.h PATHS /usr/include)
FIND_LIBRARY (POCO_FOUNDATION PocoFoundation PATHS /usr/lib)
FIND_LIBRARY (POCO_NET_LIBRARY PocoNet PATHS /usr/lib)
FIND_LIBRARY (POCO_UTIL_LIBRARY PocoUtil PATHS /usr/lib)
FIND_LIBRARY (POCO_XML_LIBRARY PocoXML PATHS /usr/lib)

IF (POCO_INCLUDE_DIR AND POCO_FOUNDATION)
   SET (POCO_FOUND TRUE)
   MESSAGE ("POCO found.")
ELSE (POCO_INCLUDE_DIR AND POCO_FOUNDATION)
   SET (POCO_FOUND FALSE)
   MESSAGE ("POCO not found...")
ENDIF (POCO_INCLUDE_DIR AND POCO_FOUNDATION)
```
  
소스 트리는 아래와 같다  
PocoTest  
 ├ CMakeLists.txt  
 ├ CMakeModules  
 │ └ FindPOCO.cmake  
 └  HelloPoco.cc    
  
  
빌드 및 실행  
```
$ cd PocoTest
$ mkdir build
$ cd build
$ cmake ..
$ make
$ ./HelloPoco
```
  