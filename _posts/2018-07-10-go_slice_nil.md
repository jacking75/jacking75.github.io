---
layout: post
title: golang - 길이가 0인 slice는 nil로
published: true
categories: [Golang]
tags: go slice nil
---
[출처](https://qiita.com/c-yan/items/5f703ac0ba87c8fb25b4)    
```
package main

import (
    "fmt"
)

func testNil(p []byte) {
    fmt.Printf("p: %v\n", p)
    fmt.Printf("len(p): %d\n", len(p))
    fmt.Printf("cap(p): %d\n", cap(p))
    for i := range p {
        fmt.Println("NG: %d\n", i)
    }
    fmt.Printf("p[:]: %v\n", p[:])
    fmt.Printf("append(p, 0): %v\n", append(p, 0))
}

func main() {
    testNil(nil)
}
```  
  
결과  
<pre>
p: []
len(p): 0
cap(p): 0
p[:]: []
append(p, 0): [0]
</pre>  
    
