---
layout: post
title: golang - slice의 shuffle
published: true
categories: [Golang]
tags: go slice shuffle
---
Fisher–Yates shuffle 이라는 알고리즘 사용  

```
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    n := 30
    arr := generate(n)

    fmt.Println(arr)
    shuffle(arr)
    fmt.Println(arr)
}

func generate(n int) []string {
    arr := make([]string, n)
    for i := range arr {
        arr[i] = fmt.Sprintf("%02d", i)
    }
    return arr
}

func shuffle(data []string) {
    n := len(data)
    for i := n - 1; i >= 0; i-- {
        j := rand.Intn(i + 1)
        data[i], data[j] = data[j], data[i]
    }
}
```
  

  
출처: https://qiita.com/sugyan/items/fd7138a756c1a409f5fd
  
