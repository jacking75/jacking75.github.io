---
layout: post
title: golang - Ubuntu에 최신 버전 설치하기
published: true
categories: [Golang]
tags: go install ubuntu linux go1.11 
---
출처: https://medium.com/@RidhamTarpara/install-go-1-11-on-ubuntu-18-04-16-04-lts-8c098c503c5f
  
### Step 1  Install Go Language  
<pre>  
sudo apt-get update
sudo apt-get -y upgrade  
</pre>  
  
  
<pre>  
cd /tmp
wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
</pre>  
  
<pre>  
sudo tar -xvf go1.11.linux-amd64.tar.gz
sudo mv go /usr/local
</pre>  
  

### Step 2 Setup Go Environment  
<pre>
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
</pre>  
  
  
### Step 3 Update current shell session
<pre>
source ~/.profile
</pre>  
  
  
### Step 4  Verify Installation
아래 명령어로 설치된 버전을 확인한다.
<pre>
go version
</pre>

go의 환경 설정 정보를 확인한다
<pre>
go env
</pre>
  