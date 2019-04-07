---
layout: post
title: C++ - Ubuntu 18 에서 g++(gcc) 설치
published: true
categories: [C++]
tags: c++ g++ gcc Ubuntu Ubuntu18 linux
---
출처: https://linuxconfig.org/how-to-install-g-the-c-compiler-on-ubuntu-18-04-bionic-beaver-linux  

우분투 18 에서 apt-get으로 설치하는 g++ 버전은 7.3 이다.  
7.3 버전은 C++17 표준을 지원한다.  
[C++ Standards Support in GCC](https://gcc.gnu.org/projects/cxx-status.html#cxx17)  

설치 명령어  
$ sudo apt install g++  
  
  
아래 명령어로 버전을 확인할 수 있다.  
$ g++ --version  
