---
layout: post
title: golang - Ubuntu에 최신 버전 설치하기
published: true
categories: [Golang]
tags: go install ubuntu linux go1.11 
---
출처: https://medium.com/@RidhamTarpara/install-go-1-11-on-ubuntu-18-04-16-04-lts-8c098c503c5f
  
### Step 1  Install Go Language  
패키지 설치  
```
sudo apt-get update  
sudo apt-get -y upgrade  
```
   
</br>  
  
직접 설치  
```
cd /tmp  
wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz  
sudo tar -xvf go1.11.linux-amd64.tar.gz  
sudo mv go /usr/local  
```  
  
  
### Step 2 Setup Go Environment
  
export GOROOT=/usr/local/go  
export GOPATH=$HOME/go  
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH  
  
  
### Step 3 Update current shell session
  
source ~/.profile  
  
  
### Step 4  Verify Installation
  
아래 명령어로 설치된 버전을 확인한다.  
go version  
  
</br>  
  
go의 환경 설정 정보를 확인한다  
```
go env  
```
   
   
# 이전 버전 제거
시스템에서 기존 Go 설치를 제거하려면 go 디렉토리를 삭제한다.   
일반적으로 Linux, macOS 및 FreeBSD에서는 `/usr/local/go`      
  
  
