---
layout: post
title: golang - CentOS 7에서 Go 설치하기
published: true
categories: [Golang]
tags: go centos linux
---
## Go 다운로드
[여기](https://golang.org/dl/ )에서 다운로드  
```
$ cd ~
$ wget https://dl.google.com/go/go1.12.6.linux-amd64.tar.gz
```
  
  
## 설치
다운로드한 파일의 업축을 푼다.  
```
$ sudo tar -C /usr/local -xzf ~/go1.12.6.linux-amd64.tar.gz
```
 
 
## 환경 변수
환경 변수에 Go의 path를 추가한다.  
```
export PATH=$PATH:/usr/local/go/bin
```
  
```
$ vim ~/.bashrc
$ source ~/.bashrc
```
  
  
## 동작 확인
  
```  
$ go version
go version go1.12.6 linux/amd64
```
  