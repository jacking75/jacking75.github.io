---
layout: post
title: CMake에서 GoogleTest 사용하기
published: true
categories: [C++]
tags: c++ cmake googletest gtest
---
[출처](https://qiita.com/imasaaki/items/c56639c86627a8a950de )  
  
  
구성은 아래와 같이 되어 있다고 한다.  
- src 폴더 아래가 테스트 대상 코드, test 폴더 아래는 테스트 코드  
    - 클래스 단위로 파일이 나누어져 있다고 가정한다.
- mk.sh을 실행하면 build/src/ 에 응용 프로그램의 실행 파일, build/test/ 에 테스트의 실행 파일이 만들어진다.  

├── CMakeLists.txt
├── README.md
├── mk.sh
├── src
│ ├── CMakeLists.txt
│ ├── ClassA.cpp
│ ├── ClassA.h
│ ├── ClassB.cpp
│ ├── ClassB.h
│ └── main.cpp
└── test
    ├── CMakeLists.txt
    ├── ClassATest.cpp
    ├── ClassBTest.cpp
    └── main.cpp
  
- CMakeLists.txt
    - googletest를 외부 프로젝트로 넣는다
        - 이용 방법은 여러 가지가 있지만,이 방법 쪽이 수동으로 설치하지 않아도 되기에 어떤 버전을 사용하고 있는지 명확하다
        - 기재 시점에서의 최신 버전을 사용(mock이 지원 되는 것은 1.8.0 에서)
        - gmock_main은 추가하지 않고, 테스트 코드도 직접 main 함수를 작성하는 것이 나중에 유용할 것 같다
    - 테스트 코드에서 응용 프로그램 코드(테스트 대상 코드)가 필요하며, 공통으로 사용하기 위해 여기에 정의해 둔다.
        - 응용 프로그램은 main.cpp 이외를 정적 라이브러리로 빌드하고, 테스트 코드에서 라이브러리를 링크시킴으로써 불필요한 컴파일을 피하는 방법이 일반적인지도 모르지만, 그러면 나중에 커버레지를 표시하려 할 때 코드가 보이지 않아 곤란하다.
  		
cmake_minimum_required ( VERSION 3.0 )  
```
cmake_minimum_required(VERSION 3.0)

#
# project settings
#
project(SIMPLE_GOOGLETEST_CMAKE_SAMPLE LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 11)


#
# find pthread for googletest
#
find_package(Threads REQUIRED)


#
# import googletest as an external project
#
include(ExternalProject)


externalproject_add(
  GoogleTest
  URL https://github.com/google/googletest/archive/release-1.8.1.zip
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}/lib
  INSTALL_COMMAND ""
  )


externalproject_get_property(GoogleTest source_dir)
include_directories(${source_dir}/googletest/include)
include_directories(${source_dir}/googlemock/include)


externalproject_get_property(GoogleTest binary_dir)
set(GTEST_LIBRARY_PATH ${binary_dir}/googlemock/gtest/${CMAKE_FIND_LIBRARY_PREFIXES}gtest.a)
set(GTEST_LIBRARY GTest::GTest)
add_library(${GTEST_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GTEST_LIBRARY} PROPERTIES
  IMPORTED_LOCATION ${GTEST_LIBRARY_PATH}
  IMPORTED_LINK_INTERFACE_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})
add_dependencies(${GTEST_LIBRARY} GoogleTest)


set(GMOCK_LIBRARY_PATH ${binary_dir}/googlemock/${CMAKE_FIND_LIBRARY_PREFIXES}gmock.a)
set(GMOCK_LIBRARY GTest::GMock)
add_library(${GMOCK_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GMOCK_LIBRARY} PROPERTIES
  IMPORTED_LOCATION ${GMOCK_LIBRARY_PATH}
  IMPORTED_LINK_INTERFACE_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})
add_dependencies(${GMOCK_LIBRARY} GoogleTest)


#
# create common variable
#
file(GLOB MY_SRCS ${PROJECT_SOURCE_DIR}/src/*.cpp)
set(MY_INCLUDES ${PROJECT_SOURCE_DIR}/src)


#
# sub directories
#
add_subdirectory(src)
add_subdirectory(test)
```
  

- src/CMakeLists.txt
    - 이곳은 특히히 해설할 부분이 없다
  
```
add_executable(${PROJECT_NAME}
  ${MY_SRCS}
  )
target_include_directories(${PROJECT_NAME} PUBLIC
  ${MY_INCLUDES}
)
```
  
- test/CMakeLists.txt
    - src 폴더 아래의 *.cpp 파일 목록에서 중복되지 않도록 src/main.cpp을 삭제한다
    - 상기 삭제한 것과 test 폴더 아래의 .cpp 파일과 .h 파일을 합쳐 모두 컴파일하게 한다
        - 이 방법은 클래스 마다 테스트 파일을 추가해도 CMakeLists.txt 편집을 할 필요가 없다
  
```
set(MY_SRCS_MINUS_MAIN ${MY_SRCS})
list(REMOVE_ITEM MY_SRCS_MINUS_MAIN ${PROJECT_SOURCE_DIR}/src/main.cpp)

file(GLOB MY_TEST_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
file(GLOB MY_TEST_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
set(MY_BINARY_NAME "UnitTestExecutor")

add_executable(${MY_BINARY_NAME}
  ${MY_TEST_SRCS}
  ${MY_SRCS_MINUS_MAIN}
  )

target_include_directories(${MY_BINARY_NAME} PUBLIC
  ${MY_INCLUDES}
  ${MY_TEST_INCLUDES}
  )

target_link_libraries(${MY_BINARY_NAME}
  GTest::GTest
  GTest::GMock
  )

add_dependencies(${MY_BINARY_NAME}
  GoogleTest
  )
```
  
  
- 각 소스 코드에 대한 설명은 생략한다. 실행한다 
  
```
$./mk.sh
$./build/test/UnitTestExectutor
```
  