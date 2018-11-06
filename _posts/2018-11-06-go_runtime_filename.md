---
layout: post
title: golang - 실행 중인 코드의 행 수와 파일 이름 얻기
published: true
categories: [Golang]
tags: go golang runtime
---
현재 실행 중인 Go의 코드 정보를 로그를 남기고 싶을 때는 코드의 파일 이름과 행 수를 알아야 한다.  
[runtime.Caller](https://golang.org/pkg/runtime/#Caller) 함수를 사용하면 이 정보들을 얻을 수 있다.  
  
```
package main

import (
  "runtime"
  "fmt"
)

func main() {
  // func Caller(skip int) (pc uintptr, file string, line int, ok bool)
  fmt.Println(runtime.Caller(1))
}
```

