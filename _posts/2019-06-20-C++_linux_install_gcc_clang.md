---
layout: post
title: Ubuntu에 gcc 8, clang 6 설치하기 
published: true
categories: [C++]
tags: c++ linux gcc clang ubuntu
---
## gcc 8 (Ubuntu 17.04)
PPA로 최신 버전 설치  
```
sudo add-apt-repository ppa:jonathonf/gcc-8.0
sudo apt update
sudo apt install gcc-8 g++-8
```
  
  
## LLVM 설치
Ubuntu 18.04  
```
apt-get install llvm
```
  
6.0 버전이 설치된다.  
  
  
## Clnag 설치
6.0 버전이 설치된다. 아래 명령어로 설치 후 연결 까지 한다.  
```
apt-get install -y clang-6.0 lldb-6.0 lld-6.0
apt-get install -y libc++-dev
ln -s /usr/bin/clang++-6.0 /usr/bin/clang++
ln -s /usr/bin/lldb-6.0 /usr/bin/lldb
ln -s /usr/bin/lld-6.0 /usr/bin/lld
```
  
