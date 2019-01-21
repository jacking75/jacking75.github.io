---
layout: post
title: LinuxBrew
published: true
categories: [OS]
tags: OS Linux brew
--- 
### 특징
- macOS의 Homebrew를 리눅스로 이식한 것
- http://linuxbrew.sh/
- root 권한이 필요 없다.
- home dir 에 설치 할 수 있다.
- 배포 package manager 에 의존하지 않는다
- 패키지 검색은 http://braumeister.org/

### 설치
#### Debian계(Ununtu)

```
$ sudo apt-get install gettext
$ sudo apt-get install build-essential curl git m4 ruby texinfo libbz2-dev libcurl4-openssl-dev libexpat-dev libncurses-dev zlib1g-dev
```
  
  
#### RHEL계(Ununtu)

```
$ sudo yum install openssl-devel
$ sudo yum groupinstall 'Development Tools' && sudo yum install curl git m4 ruby texinfo bzip2-devel curl-devel expat-devel ncurses-devel zlib-devel
```
  
  
#### install

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/linuxbrew/go/install)"
```
  
  
  
### 환경설정

```
// .bashrc or .zshrc
export PATH="$HOME/.linuxbrew/bin:$PATH"
export MANPATH="$HOME/.linuxbrew/share/man:$MANPATH"
export INFOPATH="$HOME/.linuxbrew/share/info:$INFOPATH"
export LD_LIBRARY_PATH="$HOME/.linuxbrew/lib:$LD_LIBRARY_PATH"
```
  
  

### 예 - git 설치

```
$ brew install git
$ which git
~/.linuxbrew/bin/git
$ git --version
git version 2.2.0
```
  