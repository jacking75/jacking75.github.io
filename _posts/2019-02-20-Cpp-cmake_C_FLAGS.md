---
layout: post
title: cmake에서 CMAKE_C_FLAGS에 설정한 값이 무시될 때
published: true
categories: [C++]
tags: c++ cmake C_FLAGS
---
[출처](https://qiita.com/tchofu/items/5645c6ee1b3dffc94ae0 )  
  
  
CMakeLists.txt 안에서 컴파일 옵션을 지정하려고 했는데 설정이 무시되는 문제를 만났다.  
컴파일 옵션 지정과 add_subdirectory 명령어 기술 순서가 문제였다.  


## 문제
CMakeLists.txt  
```
cmake_minimum_required(VERSION 3.12)
project(cmake_01)
set(APP_NAME example)
add_subdirectory(src)

# 여기가 무시된다.
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -O0")

set(CMAKE_CXX_FLAGS ${CMAKE_C_FLAGS})
set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
```
  
  
make 실행（-g만 지정된다)  
```
/usr/bin/c++   -I../include  -g   -o main.cpp.o -c main.cpp
/usr/bin/cc  -I../include  -g   -o target.c.o   -c target.c
```
  
  
  
## 해결
add_subdirectory 명령어를 호출 하기 전에 CMAKE_C_FLGAS를 설정한다.  

CMakeLists.txt  
```
cmake_minimum_required(VERSION 3.12)
project(cmake_01)
set(APP_NAME example)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -O0")

set(CMAKE_CXX_FLAGS ${CMAKE_C_FLAGS})
set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})

# add_subdirectory 명령어 호출 장소를 여기로 옮겼다
add_subdirectory(src)
```
  
  
make 실행( 생각한대로 옵션이 먹혔다)  
```
/usr/bin/c++   -I../include    -Wall -g -g -O0   -o main.cpp.o -c main.cpp
/usr/bin/cc  -I../include    -Wall -g -g -O0   -o target.c.o   -c target.c
```
  
