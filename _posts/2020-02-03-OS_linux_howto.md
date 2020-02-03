---
layout: post
title: Linux How To
published: true
categories: [OS]
tags: os linux howto
---
# 네트워크
  
## 특정 port에 접속한 네트워크 정보 보기
32452 port에 접속한 네트워크 정보를 보고 싶은 경우   
```
netstat -nap | grep :32452
```
  
  
  
# tar.bz2, tar.gz 파일 압축 풀기 명령어
tar.bz2 파일 압축 풀기    
```  
tar -xvf test.tar.bz2
```
  
  
tar.bz2 압축 내용 보기  
```
tar -tvf test.tar.bz2
```
  
  
tar.gz 파일 압축 풀기  
```
tar -xvzf test.tar.gz
```
  
  
  
  