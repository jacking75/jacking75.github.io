---
layout: post
title: golang - The Go Playground에서 유닛테스트 하기
published: true
categories: [Golang]
tags: go Playground unittest
---
Go는 온라인 상에서 코드를 실행 해 볼 수 있다.  
[The Go Playground](https://play.golang.org/)
  
The Go Playground에서는 유닛테스트 코드도 실행 할 수 있다.    
```
package main

import "testing"

func Hello(name string) string {
    return "Hello, Emacs!"
}

func TestHello(t *testing.T) {
    if Hello("Vim") != "Hello, Vim!" {
        t.Error("Use Vim!!!")
    }
}
```  
  
[코드 출처](https://qiita.com/cia_rana/items/b6bd6dd9f5af3f95ea2a)  
  