---
layout: post
title: git - submodule 사용 예
published: true
categories: [Git]
tags: git submodule
---
C++ 오프소스 라이브러리 googletest를 submodule로 설치 후 커밋하기  
  
```
cd helloworld
mkdir third_party
cd third_party/
git submodule add git@github.com:google/googletest.git gtest
cd gtest
git checkout release-1.8.1
git add ../../../.gitmodules
git commit 
```  
