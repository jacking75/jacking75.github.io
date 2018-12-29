---
layout: post
title: golang - Go 패키지 경로와 패키지 이름은 달라도 좋은가?
published: true
categories: [Golang]
tags: go package
---
[원문](https://qiita.com/qt-luigi/items/e1ccc82e4000a850ab1b)  
  
Go 에서는 패키지 패스의 최후의 요소는 go 파일의 패키지 이름과 같이 하는 것이 관례인데 "서로 다르면 어떻게 될까?"  

  
## 확인해 보자  

경로 트리  
<pre>
$ GOPATH /
   + - src /
      + - japan /
         + - main.go
         + - area /
            + - chugoku.go
</pre>           

     

## 변경 전 : 패키지 경로의 마지막 요소와 패키지 이름이 같다
마지막 요소 : area   
패키지 이름 : area  
  
area/chugoku.go  
```
package area

import "fmt"

func Okayama() {
    fmt.Println("피치")
}
```  
    
main.go  
```
package main

import (
    // 패키지 패스의 최후의 요소：area
    "japan/area"
)

func main() {
    // 패키지 이름：area
    area.Okayama()
}
```  
  
실행 잘 됨.  


      
## 변경 후 : 패키지 경로의 마지막 요소와 패키지 이름이 다르다
마지막 요소 : area   
패키지 이름 : chugoku  

area/chugoku.go  
```
package chugoku

import "fmt"

func Okayama() {
    fmt.Println("ピーチ")
}
```  
  
main.go  
```
package main

import (
    // 패키지 패스의 최후의 요소：변경 없음
    "japan/area"
)

func main() {
    // 페키지 이름：area -> chugoku
    chugoku.Okayama()
}
```
  
실행 잘됨.  
  
  
## 결론적으로
import 문은 패키지가 아닌 경로 이름(문자열인 것도 납득)를 지정하고 있기 때문에 패키지 경로의 마지막 요소는 아래의(go 파일) 패키지 이름과 '달라도 괜찮다'이다.  
  