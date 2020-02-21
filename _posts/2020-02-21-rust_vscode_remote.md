---
layout: post
title: Rust - VS Code의 Remote Container로 Rust를 시작하기
published: true
categories: [Visual Studio]
tags: rust vs visualstudio vscode
---
[출처](https://qiita.com/koinori/items/52523870b126697c0d28 )  
  
VS Code의 Remote Container 기능을 사용하여 최대한 빨리 Rust을 시작하려는 사람들에게 아래의 docker-compose.yaml 를 준비했다!  
  
docker-compose.yaml  
```
version: '3'
services:
    rust:
        image: rust:1.38
        volumes:
          - .:/opt/rustapp
        environment:
          - USER=`${USER}`
        working_dir: /opt/rustapp
        stdin_open: true
```
  
rust:1.38는 현재 최신 버전이다. [Docker Hub의 Rust 페이지](https://hub.docker.com/_/rust )에서 배포 되는 원하는 버전을 지정하자.  
  
`/opt/rustapp`이 적당하다. 이쪽도 프로젝트 또는 바이너리의 이름에 맞게 조정한다.  
  
`${USER}`는 `Cargo`를 실행할 때 환경 변수 `USER`가 없으면 에러가 나오므로 설정한다.  
 
 
 
## Docker Compose에서 컨테이너의 시작
이어 아래의 명령으로 위의 컨테이너를 시작한다.  
```
$ cd 프로젝트 폴더
$ docker-compose up -d
```
  
  
  
## VS Code에서 열기
VS Code를 시작하고 Remote Explorer의 Dev Containers 탭에서 `프로젝트 폴더와 같은 이름의 컨테이너`를 찾아서 `connect to container` 한다.  
컨테이너에 연결된 상태에서 VS Code의 새 창이 열린다.  
`Open Folder`에서 위 `docker-compose.yaml`의 `volumes`에서 지정 폴더(이번의 경우 `/opt/rustapp` )를 연다.  
  
  
  
## 터미널에서 Cargo 실행
VS Code에서 새 터미널을 열고 아래 명령어로 Rust 프로젝트를 시작한다.  
```
$ cargo init
     Created binary (application) package
```
  
Cargo.toml 파일과 src 폴더가 성공적으로 생성된다.  
  
src 폴더에는 샘플(?)의 `Hello World` 출력 코드가 이미 있으므로 다음의 명령으로 빌드 및 실행하자!  
```
$ cargo install --path .
  Installing rustapp v0.1.0 (/opt/rustapp)
   Compiling rustapp v0.1.0 (/opt/rustapp)
    Finished release [optimized] target(s) in 0.90s
  Installing /usr/local/cargo/bin/rustapp
   Installed package `rustapp v0.1.0 (/opt/rustapp)` (executable `rustapp`)

$ rustapp
Hello, world!
```
  
cargo install에서 폴더 이름과 동일한 이름의 바이너리를 빌드하고 `/usr/local/cargo/bin/` 아래에 설치하여준다.  
  
그리고 이 시점에서 이미 명령으로 사용할 수 있게 되어 있다!  
이번 경우 바이너리인 rustapp 이라는 명령어을 치면 Hello, World! 라고 응답한다.  
  
