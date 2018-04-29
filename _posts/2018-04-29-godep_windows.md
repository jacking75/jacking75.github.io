---
layout: post
title: golang - Windows에서 godep 설치 후 godep 명령어를 찾지 못하는 경우 
published: true
categories: [Golang]
tags: go godep
---
godep은 Go언어의 라이브러리가 의존하고 있는 패키지를 관리하는 툴로 node.js의 npm이나 Ruby의 Gem과 비슷한 것이다.  
  
godep을 Windows에서 설치 후 실행하면 아래와 같은 에러가 나오는 경우는 설치한 godep 실행 파일이 있는 디렉토리가 path에 들어가 있지 않기 때문이다.
  
![](/images/golang/godep_20171018_01.PNG)  
  
![](/images/golang/godep_20171018_02.PNG)    
  
**참고로 godep을 실행할 디렉토리는 꼭 GoPath로 설정한 디렉토리 안에 있어야 한다.**  
  