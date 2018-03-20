---
layout: post
title: C++ http 클라이언트 요청하기 - libcurl, cpp-netlib, cpprestsdk 
published: true
categories: [Network]
tags: cpp http libcurl cpp-netlib cpprestsdk
---
## libcurl
  
### 개요
- 공식 사이트. https://curl.haxx.se/libcurl/
- 이 라이브러리를 C++ 클래스로 랩핑한 프로젝트도 있다.  https://github.com/mrtazz/restclient-cpp  
    - VS2017도 지원한다(단 VS 프로젝트는 없다)
  
### 빌드
1. **vcpkg**로 입수(추천)

2. 직접 빌드한다.
- [(일어)libcurl을 windows의 msvc에서 빌드한다](http://cvl-robot.hateblo.jp/entry/2015/04/27/213924)  
- [(일어)Windows 버전 curl을 빌드해 보았다](http://hanagurotanuki.blogspot.kr/2016/08/windowscurl.html)  
- [(일어)VisualStudio 2013에서 libcurl을 build 해 보았다](http://white-raven.hatenablog.com/entry/2014/11/23/024649)  
- [(일어)libcurl을 Visual Studio 2012을 사용하여 Windows10(64bit) 상에서 빌드한 수순. 다운로드에서 빌드까지](http://microsoftcream87.hateblo.jp/entry/2017/04/22/200331)  
  
<br>  
  
## cpp-netlib
  
### 개요
- 공식 사이트. http://cpp-netlib.org
- asio 필요
- 서버와 클라이언트 양쪽  모두의 기능 제공.
  
<br>  
  
## cpprestsdk
  
### 개요
- 공식 사이트. https://github.com/Microsoft/cpprestsdk
- asio 필요.
- MS에서 제공하는 것이라서 nuget 혹은 vcpkg로 쉽게 설치 가능.
- 서버와 클라이언트 양쪽  모두의 기능 제공.  
  