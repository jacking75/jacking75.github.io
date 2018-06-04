---
layout: post
title: golang - thread pool과 GOMAXPROCS는 관계 없음
published: true
categories: [Golang]
tags: go 번역
---
golang의 경우 스레드 풀을 사용하고 있으며 블러킹 하는 시스템 콜이라면 다른 스레드로 옮겨서 처리한다. 
부족한 경우에는 OS가 허락하는 한도 내에서 만든다.  
이 수는 GOMAXPROCS 와는 관계 없다.   
  