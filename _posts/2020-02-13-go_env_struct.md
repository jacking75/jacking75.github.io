---
layout: post
title: golang - Go 언어의 응용 프로그램 설정 및 환경 변수를 Struct에 정리
published: true
categories: [Golang]
tags: go struct env
---
[출처](https://qiita.com/yuukive/items/27593cd6f3e7f264516b )  
  
## Go 언어에서 환경 변수를 읽기  
Go 언어의 표준 패키지에서 쉽게 환경 변수를 읽을 수 있다.  
  
```
fmt.Println(os.Getenv("HOME"))
```
  
그러나 때때로 환경 변수가 없을 때의 기본 설정을 할 수 있다.   
```
home := os.Getenv("HOME")
if home == "" {
  home = "/home/default/place"
}

fmt.Println(home)
```
  
  
## caarlos0/env를 사용하여 Struct에 읽기
[env](https://github.com/caarlos0/env ) 라는 라이브러리이다.   
환경 변수에서 읽어 내고 싶은 정보는 Struct에 모으고, tag에 정보를 부여하여 기본값 설정 및 필수 설정 등을 할 수 있다.  
  
```
package main

import (
    "fmt"

    "github.com/caarlos0/env"
)

type Config struct {
    Home         string   `env:"HOME"`
    Port         int      `env:"PORT" envDefault:"3000"` // 기본 값 설정
    IsProduction bool     `env:"PRODUCTION"`
    Hosts        []string `env:"HOSTS" envSeparator:":"` // 배열도 읽을 수 있다
    SecretKey    string   `env:"SECRET_KEY,required"` // 필수 설정 도 가능
    Notag        string   // 태그 없이 하면 환경 변수가 아닌 설정도 같이할 수 있다
}

func main() {
    cfg := Config{}
    env.Parse(&cfg) // 파스 하면 환경 변수에서 값이 읽어진다
    fmt.Println(cfg)
}
```
  

  