---
layout: post
title: Rust - Windows에서 scoop로 Rust 설치하기
published: true
categories: [Rust]
tags: rust install 설치 scoop
---
[출처](https://qiita.com/dozo/items/378452a0c3585f0756dc )  
  
## Scoop 설치
설치 방법은 아래의 구글링에서 나온 글을 참고한다.  
https://www.google.com/search?q=scoop+%EC%84%A4%EC%B9%98&rlz=1C1NHXL_koKR713KR713&oq=scoop+%EC%84%A4%EC%B9%98&aqs=chrome..69i57j0l4.4881j0j1&sourceid=chrome&ie=UTF-8
  
  
## gcc 설치
필요한 것을 설치한다  
```
PS C:\> scoop install pkg-config openssl gcc
PS C:\> which gcc
C:\Users\magic\scoop\apps\gcc\current\bin\gcc.EXE
PS C:\> which ar
C:\Users\magic\scoop\apps\gcc\current\bin\ar.EXE
```
  
설치 후 PowerShell을 재 실행하는 것이 좋다.  
  
  
  
## rustup 설치
Scoop 패키지에서는 rustup 과 rust 라는 Rust 관련한 것이 2개 있다.  
간단하게 말하면 rust는 일반 유저용(명령어만 있는), rustup은 개발자용이다.  
**rust가 먼저 설치 되어 있으면 rustup을 설치할 수 없다**  
  
```  
PS C:\> scoop uninstall rust # 에러가 나와도 넘어간다
PS C:\> scoop install rustup
```
  
Scoop로 설치하면 기본 환경 변수는 아래와 같다.  
```
CARGO_HOME=C:\Users\magic\scoop\persist\rustup\.cargo
RUSTUP_HOME=C:\Users\magic\scoop\persist\rustup\.rustup
```
  
  
  
## 툴 체인 설정
rust 개발에 필요한 rustc, cargo 등의 툴 군.  
이것들을 stable, beta, nightly에서 선택할 수 있지만 역시 stable을 추천한다.  
stable-gnu 와 stable-msvc가 있지만 Windows 상의 VisualStudio(VSCode가 아닌)가 설치 되어 있을 때는 msvc, 아니라면(WSL 나 VSCode, Clion 등)에서 개발할 때는 gnu를 선택한다.  
  
```
PS C:\> rustup install stable
PS C:\> rustup default stable
```  
(rustup을 넣은 시점에서 이렇게 되어 있다)
  
  
  
## 표준 라이브러리 설치
이것이 있으면 코드 보완을 할 수 있다.  
```
PS C:\> rustup component add rust-src
```
  
  
  
## 프로젝트 만들어 보기
잘 설치 되었는지 확인을 위해 Hello 프로젝트를 만들어 본다.  
```
PS C:\> cargo new hello
PS C:\> cd hello
PS C:\hello\> cargo build
PS C:\hello\> ./target/debug/hello.exe
Hello, world!
```
  
