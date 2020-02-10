---
layout: post
title: CentOS에서 프로그램의 코어 덤프 파일을 남기는 장소
published: true
categories: [OS]
tags: OS Linux CentOS Core Dump
---
[참조](https://qiita.com/sicksixrock66/items/a186afa131aa9dabfe1c )  
  
어떤 이유로 프로그램이 오류로 비정상 종료하는 경우 기본적으로 프로세스의 현재 디렉토리에 core.PID 형태로 덤프를 남긴다.  
  
CentOS에서는 기본적으로 코어 덤프를 남기지 않게 설정 되어 있다.  코어 덤프를 남기고 싶다면 아래처럼 수정해야 한다.  
```
vi /etc/systemd/system.conf
```  
```
DumpCore=yes
DefaultLimitCORE=infinity
```
  
위 설정으로 코어 덤프 파일을 남기고, 코어 덤프 파일의 최대 크기를 무제한이 된다.  
  

