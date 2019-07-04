---
layout: post
title: LinuxBrew 설치하기 (CentOS, WSL)
published: true
categories: [OS]
tags: OS Linux brew LinuxBrew wsl
---
## Linux (CentOS)에 Homebrew on Linux (Linuxbrew)를 설치하기 전에
출처: https://qiita.com/libra_lt/items/548fcbdfbcf992cba4ed  
  
CentOS Linux release 7.6.1810 (Core)  
  
## Linuxbrew는
다음 특징을 가진 패키지 관리 도구이다. 
- root가 아닌 사용자의 home에 설치하는 것이 가능하다. sudo가 필요 없다. 
- 호스트 배포판에 패키지 되지 않은 소프트웨어를 설치할 수 있다.
- 호스트 배포판이 오래된 경우 최신 버전을 설치할 수 있다.
  
  
## 설치
### 전제
Linux 환경 구축하고, root가 아닌 사용자를 작성하는 것.   
  
  
### Linuxbrew 필수 패키지를 설치
Linuxbrew를 설치하기 전에 필요한 패키지를 설치한다.  
  
```
sudo yum groupinstall 'Development Tools' && sudo yum install curl file git ruby which
```
  
  
### Linuxbrew 설치
  
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"
```
  
설치 시 어디에 설치하는지 확인 되지만, 권장된 /home/linubrew에 설정했다.  
  
  
### 경로 설정
bash 이용자는 bash 설정 파일에 경로를 추가한다. 여기에서는 bash_profile에 경로를 추가한다.  
  
```
echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >>~/.bash_profile
eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```
  
  
### 확인
"Hello, world!"라고 표시만 하는 패키지를 설치한다.   
hello 명령을 입력하고 실행 할 수 있는지 확인한다.  
  
```
brew install hello
```
  
  
  
## Widnwos의 개발 환경을 정리 : WSL + Linuxbrew 도입
출처: https://qiita.com/kentom/items/e486cc7f85c8f3187d4e  
  
## 환경
윈도우 10  
WSL : 우분투 18.04  
Linuxbrew 2.1.0  
  
  
## Linuxbrew 도입하기
공식 사이트에 나와 있는대로 설치하면 좋다   
  
```
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"
```
  
만약 Warning: /home/linuxbrew/.linuxbrew/bin is not in your PATH. 라는 경고 메시자가 나왔다면 아래를 ~/.bashrc 에 추가  
  
```
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```
  
source ~/.bashrc 하여 WSL 재시작 하고, .bashrc 파일을 로딩 했다면 brew 명령어를 사용할 수 있게 되었으므로 공식 사이트처럼 Hello World를 설치해 본다.  
  