---
layout: post
title: golang - 랜덤 seed 설정하기
published: true
categories: [Golang]
tags: go random rand seed
---
출처: https://qiita.com/makiuchi-d/items/9c4af327bc8502cdcdce
  
  
## 시간을 seed로 설정하기
time.Now().UnixNano()를 사용한다.  
  
```
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    fmt.Println(rand.Int63())
}
```
  
  
  
## 보안성이 높은 seed 사용하기
crypto/rand 패키지를 사용한다.  
  
```
package main

import (
    crand "crypto/rand"
    "fmt"
    "math"
    "math/big"
    "math/rand"
)

func main() {
    seed, _ := crand.Int(crand.Reader, big.NewInt(math.MaxInt64))
    rand.Seed(seed.Int64())
    fmt.Println(rand.Int63())
}
```
  
  
  
## 메르센 트위스터 난수 생성기 사용하기 
오픈 소스로 공개된 것을 사용한다.  
https://github.com/seehuhn/mt19937  
  
```
package main

import (
    crand "crypto/rand"
    "fmt"
    "math"
    "math/big"
    "math/rand"

    "github.com/seehuhn/mt19937"
)

func main() {
    seed, _ := crand.Int(crand.Reader, big.NewInt(math.MaxInt64))
    rng := rand.New(mt19937.New())
    rng.Seed(seed.Int64())
    fmt.Println(rng.Int63())
}
``` 