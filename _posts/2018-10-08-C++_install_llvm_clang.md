---
layout: post
title: C++ - Ubuntu에 clang, libc++-, lldb 설치하기
published: true
categories: [C++]
tags: c++ Ubuntu clang lldb linux
---

```
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
echo deb http://apt.llvm.org/stretch/ llvm-toolchain-stretch-6.0 main>> /etc/apt/sources.list
echo deb-src http://apt.llvm.org/stretch/ llvm-toolchain-stretch-6.0 main >> /etc/apt/sources.list
sudo apt-get update
sudo apt-get install -y clang-6.0 lldb-6.0 lld-6.0
sudo apt-get install -y libc++-dev
sudo ln -s /usr/bin/clang++-6.0 /usr/bin/clang++
sudo ln -s /usr/bin/lldb-6.0 /usr/bin/lldb
sudo ln -s /usr/bin/lld-6.0 /usr/bin/lld
```