---
layout: post
title: 내 PC에서 실행 중인 VirtualBox에 내트워크 연결하기
published: true
categories: [Network]
tags: network virtualbox
--- 
내 PC에서 실행 중인 VirtualBox에 설치된 애플리케이션에 네트워크로 접속하려면 VirtualBox의 '포트 포워딩'을 설정하면 된다.  
    
아래는 VirtualBox에 설치된 MongoDB를 내 PC에서 접속하기 위해 포트 포워딩 설정을 하였다.  
    
VirtualBox에 설치된 우분투의 네트워크 설정은 아래와 같다.  
![](/images/virtualbox/001.PNG)    
    
  
아래의 화면에서 포트 포워딩 설정 창을 연다.  
![](/images/virtualbox/002.PNG)  
  
127.0.0.1:27017 로 접속을 시도하면 주소를 10.0.2.15:27017로 포워딩 하도록 한다.  
![](/images/virtualbox/003.PNG)  
  