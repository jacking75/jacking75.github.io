---
layout: post
title: golang - package의 prefix를 생략
published: true
categories: [Golang]
tags: go package
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
```
// main.go
package main

import (
  "log"
)

func main() {
  log.Printf("hello")
}
```  
에서 "log."를 생략하려면  
```
import (
  . "log"
)
```  
로 하면  
```
Printf("hello")
```  
라고 쓸수 있다. 마찬가지로,  
```
import (
  foo "log"
)
```  
라고 하면  
```
foo.Printf("hello")
```  
로 접근 할 수 있다  
  
  
