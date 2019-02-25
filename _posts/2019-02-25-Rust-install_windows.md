---
layout: post
title: Windows에 Rust 설치하기
published: true
categories: [Rust]
tags: rust install
---
Rust를 Windows에서 사용할 때 C++ 컴파일러가 별도로 필요하므로 먼저 VisualStudio Build Tools를 설치한다.(Visual Studio를 설치하면 된다)  
   
## Rustup 설치
[Rustup](https://rustup.rs/) 사이트에 가서 rustup‑init.exe 링크를 클릭해서 다운로드 한다.  
다운로드한 rustup-init.exe을 클리하여 설치한다.  
  
설치가 끝난 후 아래 명령어를 실행하여 잘 설치 되었는지 확인한다.  
```
rustc --version
```
  
  
## WSL에 설치하기
linux 설치와 비슷하다.  
```
curl https://sh.rustup.rs -sSf | sh
```
   
