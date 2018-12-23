---
layout: post
title: AWS Lambda에서 C++ 프로그램 실행하기
published: true
categories: [C++]
tags: c++ aws lambda
---
[이 글](https://qiita.com/kyamamoto9120/items/bc487688184f13b0c5e2 )을 번역한 것이다.  

re:Invent 2018에서 AWS Lambda의 새로운 기능, Custom Runtime이 발표 되었다.  
이 기능은 Custom Runtime을 준비하는 것으로서 임의의 언어로 Lambda를 사용할 수 있다.  

Custom Runtime은 직접 구현해도 괜찮고, AWS가 제공하는 것도 있다.  
아래는 2018/11/30 18:00 시점에 공개되어 있는 Custom Runtime 이다.   
- https : // github. co.kr / awslabs / aws-lambda-cpp  
- https://github.com/awslabs/aws-lambda-rust-runtime  
  
이 글에서는 공개된 C++의 Custom Runtime을 사용하여 C++로 구현한 Lambda 함수를 작성한다.  

<br/>
<br/>
  
## Runtime 만들기
기본적으로는 aws-lambda-cpp에 기록된 순서를 따른다.  
빌드는 리눅스에서 해야하기 때문에 여기에서는 아래와 같은 도커를 준비하였다.  

Dockerfile
```
FROM alpine:latest

RUN apk update && apk add cmake make git g++ bash curl-dev zlib-dev zip

WORKDIR /usr
RUN git clone https://github.com/awslabs/aws-lambda-cpp-runtime.git && \
  cd aws-lambda-cpp-runtime && \
  mkdir build && \
  cd build && \
  cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=~/lambda-install && \
  make && make install

WORKDIR /home
```
  
아래처럼 빌드, 시작한다.  
```
$ docker build -t aws_lambda_cpp .
$ docker run --rm -it -v `pwd`:/home aws_lambda_cpp:latest /bin/bash
```
  
  
  
## Lambda 함수를 만든다
여기도 튜토리얼에 따른다.  

아래 2개의 파일을 준비한다.  
CMakeLists.txt  
```
cmake_minimum_required(VERSION 3.5)
set(CMAKE_CXX_STANDARD 11)
project(demo LANGUAGES CXX)
find_package(aws-lambda-runtime)
add_executable(${PROJECT_NAME} "main.cpp")
target_link_libraries(${PROJECT_NAME} PRIVATE AWS::aws-lambda-runtime)
target_compile_features(${PROJECT_NAME} PRIVATE "cxx_std_11")
target_compile_options(${PROJECT_NAME} PRIVATE "-Wall" "-Wextra")

# this line creates a target that packages your binary and zips it up
aws_lambda_package_target(${PROJECT_NAME})
```  

main.cpp  
```
#include <aws/lambda-runtime/runtime.h>

using namespace aws::lambda_runtime;

static invocation_response my_handler(invocation_request const& req)
{
  if (req.payload.length() > 42) {
    return invocation_response::failure("error message here"/*error_message*/,
                                        "error type here" /*error_type*/);
  }

  return invocation_response::success("json payload here" /*payload*/,
                                      "application/json" /*MIME type*/);
}

int main()
{
  run_handler(my_handler);
  return 0;
}
```
  
Docker 컨테이너 내에서 아래처럼 실행한다.  
```
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=~/lambda-install
$ make
$ make aws-lambda-package-demo
```
  
  
배포한다. Role은 적절하게 설정한다.  
```
$ aws lambda create-function --function-name demo \
--role <specify role arn from previous step here> \
--runtime provided --timeout 15 --memory-size 128 \
--handler demo --zip-file fileb://demo.zip
```
  
배포에 성공하였다면 Lambda 함수를 실행한다.  
```
$ aws lambda invoke --function-name demo --payload '{"answer":42}' output.txt
```
  
  
정상으로 동작하였다면 아래처럼 표시될 것이다.  
표준 출력  
```
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```  
output.txt  
```
json payload here
```  

