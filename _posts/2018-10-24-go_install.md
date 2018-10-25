---
layout: post
title: golang - 설치와 GoPath
published: true
categories: [Golang]
tags: go golang install gopath
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
# 설치
- [(일어)Ubuntu에 GO 언어(golang)의 최신 버전을 apt로 설치](http://qiita.com/matsumode/items/ffe810b0c3f788d1a4e5)
- [(일어)Homebrew로 Go 1.6 설치](http://qiita.com/minoritea/items/9565c02b1931495a1f4f) 
- [(일어)Go 언어 개발환경 셋업](http://qiita.com/awakia/items/7bf03fd96a74502073b8) 
- [(일어)Go 개발 환경/빌드 환경으로서 Amazon Linux을 셋업하는 절차](http://dev.classmethod.jp/cloud/aws/amazon-linux-go-setup/) 
- [(일어)CentOS7에 Go 최신 버전을 설치하기](https://qiita.com/nekootoko3/items/7004b19ff7adffb58b39)  
  
  
## govirtualenv
- windows, linux, mac 모두 지원
- [(일어)govirtualenv: Go 언어용 python virtualenv](https://qiita.com/necomeshi/items/ab14b1f615e5066475cc)
  
    
## goenv
- goenv는 rbenv과 마찬가지로 golang 버전을 관리할 때 사용하는 것.
    - https://github.com/wfarr/goenv
- goenv를 설치한다  

```
git clone https://github.com/wfarr/goenv.git ~/.goenv
```  
  
- 패스 설정  

```
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
eval "$(goenv init -)"
```
  
- golang 설치  

```
goenv install 1.4
goenv versions
```
  
- rbenv과 마찬가지로 golang을 사용하도록 설정  

```
goenv global 1.4
go version
```
  
- 참고
    - [(일어)goenv로 gae/go와 보통의 go 환경을 바꾼다](http://qiita.com/sinmetal/items/71cfba4ae27cc2366572)  

## gobrew
- [공식 사이트](https://github.com/cryptojuice/gobrew)
- 설치  

```
curl -L https://raw.github.com/grobins2/gobrew/master/tools/install.sh | sh
```
  

## GVM
- ruby로 말하면 rbenv 같은 것  
    - https://github.com/moovweb/gvm  
  
- 설치

```
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```  

    - ~/.gvm에 저장소가 만들어진다.
- bashrc에서 gvm 설정 읽기

```
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
``` 

    - 이로써 source~/.bashrc를 하면 gvm을 실행할 수 있게 된다.
  
- go 설치

```
gvm listall
gvm use go1.4.2
```
  
- 확인한다

```
$ gvm list
gvm gos (installed)
   go1.4.2
   system
```
  
- 사용

```
$ go version
go version go1.4.2 linux/amd64
```
  
- 여담

```
gvm use go1.4.2
```  
라고 ~/.bashrc에 추가해 두면 매번 로그인 시점에서 지정한 버전의 go를 사용할 수 있게 된다.
    
  
- [Golang GVM 사용하기](http://itrepreneur.tistory.com/11) 
- [(일어)GVM을 사용하여 Go 언어를 Mac에 설치](http://qiita.com/kitsuki00/items/91c29b147dbf5577cccb) 
- [(일어)Go 개발 환경을 셋업](http://golang.rdy.jp/2017/10/25/environment/) 
- [(일어)GVM을 사용하여 복수 버전의 Go를 편리하게 교체하자](https://qiita.com/reoring/items/7344399ca6db99d2746f) 
- [(일어)gvm을 이용하여 Revel 개발환경을 만들자](http://blog.mnrtks.jp/posts/2014/02/24/gvm-revel/)   
    
      
## GoPath
- $GOPATH는 원하는대로 지정할 수 있다
- go 공식문서에서는 go 만드는 프로젝트를 GOPATH 아래에 두는 것을 추천한다.
- Go 패스는 import문의 해결에 사용된다. go/build 패키지에서 구현하고 있다.
- 환경 변수 GOPATH는 Go 코드를 찾을 장소를 뜻한다.(PATH 같은 것）
    - Unix의 경우는 콜럼으로 구별한다.
    - Windows의 경우는 세미콜롬으로 구별한다.
- GOPATH는 표준 Go 밖에서 패키지를 빌드 하거나 설치할 때에 설정되어 있지 않으면 안된다.
- GOPATH 에 들어가 있는 디렉토리는 각각 정해진 구조를 가지고 있어야 한다.  
예:

<pre>
    GOPATH=/home/user/gocode

    /home/user/gocode/
        src/
            foo/
                bar/               (go code in package bar)
                    x.go
                quux/              (go code in package main)
                    y.go
        bin/
            quux                   (installed command)
        pkg/
            linux_amd64/
                foo/
                    bar.a          (installed package object)
</pre>  
 
- Go는 GOPATH의 각 디렉토리에서 소스코드를 찾지만 다운로드한 파일은 보통 GOPATH  중에서 가장 최초의 디렉토리에 설치한다.
  

### 복수의 GoPath 지정 
- 가능하다. 단 첫번째 지정한 것 이외는 넘어가는 경우가 있다.
- 예를들면 아래와 같이 GOPATH를 지정하였다.  
    $ export GOPATH=$HOME/go:$HOME/golang:$HOME/workspace/go  
- 지정한 모든 GOPATH가 유효한 것은 Go 컴파일러가 외부의 패키지를 찾는 경우만이다.
- 즉 import 문에서 지정한 패키지는 GOPATH가 통하는 곳에 있다면 컴파일러가 발견할 수 있다(컴파일러 에러가 되지 않는다).  
- $HOME/workspace/go/src 이하에 배치한 패키지는 imoort 할 수 없다.
- 역으로 첫번째 GOPATH만 유효한 경우는 go get 에 의해 외부의 패키지를 다운로드・설치 하는 경우이다.
- go get 명령어를 사용하면 외부의 패키지는 꼭 첫번째에 지정한 $HOME/go 에 설치된다.  
  
- [(일어)GoPath와 작업 디렉토리](https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/01.2.md)
- [(일어)GOPATH 오염 문제](http://qiita.com/spiegel-im-spiegel/items/73ebc684b5807277b7e2) 
- [(일어)외부 모듈을 import 하기 전에 필요한 GOPATH 설정하는 것에 대해서 여러가지 조사](http://qiita.com/HirofumiYashima/items/20a33dd10da437bb1277) 


